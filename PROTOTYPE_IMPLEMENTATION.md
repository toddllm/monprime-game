# MonPrime - Prototype Implementation Guide

## Quick Start: Building the First Playable Demo

This guide provides concrete implementation steps for creating a MonPrime prototype in Unity with AR capabilities.

## Project Setup

### 1. Unity Project Configuration

```bash
# Unity 2023.2 LTS recommended
# Required packages (install via Package Manager):
- AR Foundation 5.0+
- ARCore XR Plugin (Android)
- ARKit XR Plugin (iOS)
- Universal Render Pipeline (URP)
- TextMeshPro
- Addressables
- Mirror Networking
```

### 2. Project Structure

```
Assets/
├── Scripts/
│   ├── AR/
│   │   ├── ARMonSpawner.cs
│   │   ├── ARPlaneManager.cs
│   │   └── ARSessionController.cs
│   ├── Combat/
│   │   ├── CombatEngine.cs
│   │   ├── DamageCalculator.cs
│   │   └── PhysicsImpact.cs
│   ├── Mons/
│   │   ├── Mon.cs
│   │   ├── MonDatabase.cs
│   │   └── MonBehavior.cs
│   ├── Player/
│   │   ├── PlayerController.cs
│   │   ├── GoldenBook.cs
│   │   └── Inventory.cs
│   ├── Systems/
│   │   ├── CurseManager.cs
│   │   ├── EnvironmentDetector.cs
│   │   └── SaveSystem.cs
│   └── Network/
│       ├── NetworkManager.cs
│       └── SyncComponents.cs
├── Prefabs/
│   ├── Mons/
│   ├── UI/
│   └── Effects/
├── Materials/
├── Textures/
└── Sounds/
```

## Core Implementation

### 1. Basic Mon Class

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

[Serializable]
public class Mon : MonoBehaviour
{
    [Header("Identity")]
    public int monId;
    public string monName;
    public MonType primaryType;
    public MonType secondaryType;
    
    [Header("Stats")]
    public float maxHealth = 100f;
    public float currentHealth;
    public float attack = 10f;
    public float defense = 5f;
    public float speed = 1f;
    public float size = 1f; // Affects if fits in book
    
    [Header("Combat")]
    public List<MonAbility> abilities;
    public bool canVaporizeHumans = false;
    public bool canCounterPunch = false;
    
    [Header("Capture")]
    public float captureRate = 0.3f;
    public BookType requiredBookType = BookType.Standard;
    
    private Rigidbody rb;
    private Animator animator;
    private bool isWild = true;
    private Player owner;
    
    void Awake()
    {
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();
        currentHealth = maxHealth;
    }
    
    public void TakeDamage(float damage, Vector3 impactDirection)
    {
        // Calculate actual damage
        float actualDamage = Mathf.Max(1, damage - defense);
        currentHealth -= actualDamage;
        
        // Visual feedback - fling the Mon
        if (rb != null)
        {
            rb.AddForce(impactDirection * damage * 10f, ForceMode.Impulse);
            StartCoroutine(RecoverFromImpact());
        }
        
        // Damage number popup
        DamagePopup.Create(transform.position, actualDamage);
        
        // Check for defeat
        if (currentHealth <= 0)
        {
            OnDefeated();
        }
        else if (canCounterPunch && UnityEngine.Random.value < 0.3f)
        {
            PerformCounterAttack();
        }
    }
    
    IEnumerator RecoverFromImpact()
    {
        yield return new WaitForSeconds(1f);
        
        // Return to fighting stance
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;
        transform.rotation = Quaternion.identity;
    }
    
    void PerformCounterAttack()
    {
        if (canVaporizeHumans && UnityEngine.Random.value < 0.1f)
        {
            // Vaporize attack
            PlayerController.Instance.Vaporize();
        }
        else
        {
            // Regular counter punch
            float counterDamage = attack * 1.5f;
            PlayerController.Instance.TakeDamage(counterDamage);
        }
    }
    
    void OnDefeated()
    {
        // Drop rewards
        int coins = UnityEngine.Random.Range(10, 50);
        int xp = UnityEngine.Random.Range(20, 100);
        
        PlayerController.Instance.AddCoins(coins);
        PlayerController.Instance.AddExperience(xp);
        
        // Capture opportunity
        if (isWild)
        {
            CaptureUI.Instance.ShowCaptureOption(this);
        }
        
        // Defeat animation
        StartCoroutine(DefeatSequence());
    }
    
    public bool AttemptCapture(GoldenBook book)
    {
        if (book.Type < requiredBookType)
        {
            Debug.Log("Book type too weak!");
            return false;
        }
        
        if (size > book.RemainingCapacity)
        {
            Debug.Log("Mon doesn't fit in book!");
            return false;
        }
        
        float modifiedCaptureRate = captureRate * (1f - currentHealth / maxHealth);
        bool captured = UnityEngine.Random.value < modifiedCaptureRate;
        
        if (captured)
        {
            isWild = false;
            owner = PlayerController.Instance;
            book.AddMon(this);
            gameObject.SetActive(false); // Remove from world
        }
        
        return captured;
    }
}

public enum MonType
{
    Normal, Fire, Water, Electric, Grass, Ice, Fighting,
    Poison, Ground, Flying, Psychic, Bug, Rock, Ghost,
    Dragon, Dark, Steel, Fairy, Void, Creator
}

public enum BookType
{
    Starter,    // 10 capacity
    Standard,   // 50 capacity
    Advanced,   // 100 capacity
    Master      // Unlimited, can capture books
}
```

### 2. AR Mon Spawning System

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using System.Collections;
using System.Collections.Generic;

public class ARMonSpawner : MonoBehaviour
{
    [Header("AR Components")]
    public ARRaycastManager arRaycastManager;
    public ARPlaneManager arPlaneManager;
    
    [Header("Spawning")]
    public GameObject[] monPrefabs;
    public float spawnRadius = 10f;
    public int maxMonsAtOnce = 5;
    
    private List<Mon> activeMons = new List<Mon>();
    private EnvironmentDetector envDetector;
    private LocationService locationService;
    
    void Start()
    {
        envDetector = GetComponent<EnvironmentDetector>();
        locationService = GetComponent<LocationService>();
        
        StartCoroutine(SpawnLoop());
    }
    
    IEnumerator SpawnLoop()
    {
        yield return new WaitForSeconds(3f); // Initial delay
        
        while (true)
        {
            if (activeMons.Count < maxMonsAtOnce)
            {
                yield return TrySpawnMon();
            }
            
            yield return new WaitForSeconds(5f); // Check every 5 seconds
        }
    }
    
    IEnumerator TrySpawnMon()
    {
        // Get spawn parameters
        var environment = envDetector.GetCurrentEnvironment();
        var weather = yield return WeatherAPI.GetCurrentWeather();
        var timeOfDay = GetTimeOfDay();
        var activeeCurse = CurseManager.Instance.ActiveCurse;
        
        // Determine which Mon to spawn
        var spawnablemons = MonDatabase.GetSpawnableMons(
            environment, weather, timeOfDay, activeeCurse
        );
        
        if (spawnablemons.Count == 0) yield break;
        
        // Select Mon based on rarity
        var selectedMon = SelectMonByRarity(spawnablemons);
        
        // Find spawn position
        Vector3 spawnPos = GetSpawnPosition();
        
        if (spawnPos != Vector3.zero)
        {
            SpawnMon(selectedMon, spawnPos);
        }
    }
    
    Vector3 GetSpawnPosition()
    {
        // Try to find AR plane
        List<ARRaycastHit> hits = new List<ARRaycastHit>();
        Vector3 randomPoint = Random.insideUnitCircle * spawnRadius;
        randomPoint = Camera.main.transform.position + 
                     new Vector3(randomPoint.x, 0, randomPoint.y);
        
        Ray ray = new Ray(randomPoint + Vector3.up * 10f, Vector3.down);
        
        if (arRaycastManager.Raycast(ray, hits, 
            UnityEngine.XR.ARSubsystems.TrackableType.PlaneWithinPolygon))
        {
            return hits[0].pose.position;
        }
        
        return Vector3.zero;
    }
    
    void SpawnMon(MonData monData, Vector3 position)
    {
        // Instantiate Mon
        GameObject monGO = Instantiate(monData.prefab, position, 
                                      Quaternion.identity);
        Mon mon = monGO.GetComponent<Mon>();
        
        // Initialize Mon data
        mon.monId = monData.id;
        mon.monName = monData.name;
        mon.primaryType = monData.primaryType;
        // ... set other properties
        
        // Add spawn effect
        GameObject spawnEffect = Instantiate(spawnEffectPrefab, position, 
                                           Quaternion.identity);
        Destroy(spawnEffect, 2f);
        
        // Track active Mon
        activeMons.Add(mon);
        
        // Alert player
        NotificationUI.Show($"A wild {mon.monName} appeared!");
        
        // Start battle requirement timer
        StartCoroutine(ForceBattleTimer(mon));
    }
    
    IEnumerator ForceBattleTimer(Mon mon)
    {
        yield return new WaitForSeconds(30f); // 30 seconds to engage
        
        if (mon != null && !mon.IsInCombat)
        {
            // Mon attacks player for avoiding battle
            mon.ForceAttackPlayer();
            NotificationUI.ShowWarning("The Mon attacked because you didn't engage!");
        }
    }
}
```

### 3. Combat System

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class CombatEngine : MonoBehaviour
{
    [Header("Combat Settings")]
    public float punchReach = 3f;
    public float punchForce = 50f;
    public LayerMask monLayer;
    
    private Camera arCamera;
    private PlayerController player;
    private float lastPunchTime;
    private float punchCooldown = 0.5f;
    
    void Start()
    {
        arCamera = Camera.main;
        player = PlayerController.Instance;
    }
    
    void Update()
    {
        HandleTouchInput();
    }
    
    void HandleTouchInput()
    {
        if (Input.touchCount > 0 && !EventSystem.current.IsPointerOverGameObject())
        {
            Touch touch = Input.GetTouch(0);
            
            if (touch.phase == TouchPhase.Began)
            {
                TryPunch(touch.position);
            }
            else if (touch.phase == TouchPhase.Moved)
            {
                // Track swipe for special attacks
                TrackSwipeGesture(touch);
            }
        }
    }
    
    void TryPunch(Vector2 screenPosition)
    {
        if (Time.time - lastPunchTime < punchCooldown) return;
        
        Ray ray = arCamera.ScreenPointToRay(screenPosition);
        RaycastHit hit;
        
        if (Physics.Raycast(ray, out hit, punchReach, monLayer))
        {
            Mon targetMon = hit.collider.GetComponent<Mon>();
            if (targetMon != null)
            {
                PerformPunch(targetMon, ray.direction);
            }
        }
        else
        {
            // Air punch animation
            player.PlayPunchAnimation();
        }
        
        lastPunchTime = Time.time;
    }
    
    void PerformPunch(Mon target, Vector3 direction)
    {
        // Calculate damage
        float baseDamage = player.AttackPower;
        float criticalChance = player.CriticalRate;
        bool isCritical = Random.value < criticalChance;
        
        float damage = baseDamage;
        if (isCritical)
        {
            damage *= 2f;
            UIEffects.ShowCritical();
        }
        
        // Apply type effectiveness
        float typeMultiplier = TypeChart.GetEffectiveness(
            player.AttackType, target.primaryType
        );
        damage *= typeMultiplier;
        
        // Deal damage with physics impact
        Vector3 impactForce = direction * punchForce;
        target.TakeDamage(damage, impactForce);
        
        // Visual feedback
        GameObject punchEffect = Instantiate(punchEffectPrefab, 
                                           hit.point, Quaternion.identity);
        Destroy(punchEffect, 1f);
        
        // Haptic feedback
        Handheld.Vibrate();
        
        // Update combat state
        CombatManager.Instance.RegisterHit(player, target, damage);
    }
    
    void TrackSwipeGesture(Touch touch)
    {
        // Implement swipe recognition for special moves
        // Up swipe = Uppercut
        // Down swipe = Slam
        // Circle = Spin attack
        // etc.
    }
}
```

### 4. Golden Book System

```csharp
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class GoldenBook : ScriptableObject
{
    [Header("Book Properties")]
    public string bookName = "Standard Golden Book";
    public BookType bookType = BookType.Standard;
    public int maxCapacity = 50;
    public Sprite bookIcon;
    
    [Header("Special Properties")]
    public bool canCaptureBooks = false; // Only Master Book
    public float captureBonus = 1f;
    
    [SerializeField]
    private List<Mon> capturedMons = new List<Mon>();
    
    public int RemainingCapacity => maxCapacity - GetTotalSize();
    
    public bool TryCaptureMon(Mon mon)
    {
        // Check if mon is defeated
        if (mon.currentHealth > 0)
        {
            Debug.Log("Mon must be defeated first!");
            return false;
        }
        
        // Check book type requirement
        if (bookType < mon.requiredBookType)
        {
            UINotification.Show($"Need {mon.requiredBookType} book or higher!");
            return false;
        }
        
        // Check capacity
        if (mon.size > RemainingCapacity)
        {
            UINotification.Show("Mon doesn't fit in book!");
            ShakeAnimation.Play(transform); // Visual feedback
            return false;
        }
        
        // Attempt capture with animation
        StartCoroutine(CaptureSequence(mon));
        return true;
    }
    
    IEnumerator CaptureSequence(Mon mon)
    {
        // Show book opening animation
        BookAnimator.PlayOpen(this);
        yield return new WaitForSeconds(0.5f);
        
        // Golden capture beam effect
        GameObject beam = Instantiate(captureBeamPrefab, 
                                    transform.position, 
                                    Quaternion.LookRotation(mon.transform.position));
        
        // Calculate capture success
        float captureChance = mon.captureRate * captureBonus;
        captureChance *= (1f - mon.currentHealth / mon.maxHealth); // Weaker = easier
        
        // Apply curse modifiers
        if (CurseManager.Instance.ActiveCurse == CurseType.HarderCapture)
        {
            captureChance *= 0.5f;
        }
        
        yield return new WaitForSeconds(1.5f); // Suspense!
        
        bool success = Random.value < captureChance;
        
        if (success)
        {
            // Success effects
            mon.transform.DOScale(0.1f, 0.5f);
            yield return new WaitForSeconds(0.5f);
            
            // Add to book
            capturedMons.Add(mon);
            mon.SetOwner(PlayerController.Instance);
            
            // Remove from world
            Destroy(mon.gameObject);
            
            UINotification.ShowSuccess($"{mon.monName} was captured!");
            SoundManager.PlayCaputureSuccess();
        }
        else
        {
            // Failure effects
            BookAnimator.PlayReject(this);
            mon.BreakFree();
            
            UINotification.Show($"{mon.monName} broke free!");
            SoundManager.PlayCaptureFail();
        }
        
        Destroy(beam);
        BookAnimator.PlayClose(this);
    }
    
    public void DeployMon(int index, Vector3 position)
    {
        if (index >= capturedMons.Count) return;
        
        Mon mon = capturedMons[index];
        GameObject deployed = Instantiate(mon.gameObject, position, 
                                        Quaternion.identity);
        
        Mon deployedMon = deployed.GetComponent<Mon>();
        deployedMon.SetAsDeployed();
        deployedMon.owner = PlayerController.Instance;
        
        // Remove from book temporarily
        capturedMons.RemoveAt(index);
        ActiveTeam.Add(deployedMon);
    }
    
    int GetTotalSize()
    {
        float totalSize = 0;
        foreach (var mon in capturedMons)
        {
            totalSize += mon.size;
        }
        return Mathf.CeilToInt(totalSize);
    }
}

// Special Master Book that can capture other books
public class MasterBook : GoldenBook
{
    private List<GoldenBook> capturedBooks = new List<GoldenBook>();
    
    public bool TryCaptureBook(GoldenBook targetBook)
    {
        if (!canCaptureBooks) return false;
        
        // Only Zeldina or special conditions
        if (!PlayerController.Instance.HasZeldinaPower)
        {
            UINotification.Show("Only Zeldina can capture books!");
            return false;
        }
        
        StartCoroutine(BookCaptureSequence(targetBook));
        return true;
    }
    
    IEnumerator BookCaptureSequence(GoldenBook targetBook)
    {
        // Epic book vs book battle animation
        yield return new WaitForSeconds(3f);
        
        capturedBooks.Add(targetBook);
        // Transfer all mons from captured book
        foreach (var mon in targetBook.capturedMons)
        {
            capturedMons.Add(mon);
        }
    }
}
```

### 5. Curse System Implementation

```csharp
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class CurseManager : MonoBehaviour
{
    public static CurseManager Instance;
    
    [Header("Curse Settings")]
    public float curseDuration = 20f;
    public float restDuration = 20f;
    
    public CurseType ActiveCurse { get; private set; }
    public float TimeRemaining { get; private set; }
    
    private List<CurseType> availableCurses;
    private Coroutine curseCoroutine;
    
    void Awake()
    {
        Instance = this;
        InitializeCurses();
    }
    
    void InitializeCurses()
    {
        availableCurses = new List<CurseType>
        {
            CurseType.DoubleSpawns,
            CurseType.PowerfulMons,
            CurseType.WeakenedPlayer,
            CurseType.NoCapture,
            CurseType.Invisibility,
            CurseType.SpeedBoost,
            CurseType.HealthDrain,
            CurseType.ReverseControls,
            CurseType.GiantMons,
            CurseType.SwarmMode
        };
    }
    
    void Start()
    {
        curseCoroutine = StartCoroutine(CurseCycle());
    }
    
    IEnumerator CurseCycle()
    {
        while (true)
        {
            // Rest period
            ActiveCurse = CurseType.None;
            UIManager.Instance.ShowCurseStatus("Rest Period", restDuration);
            
            TimeRemaining = restDuration;
            while (TimeRemaining > 0)
            {
                yield return new WaitForSeconds(1f);
                TimeRemaining--;
                UIManager.Instance.UpdateCurseTimer(TimeRemaining);
            }
            
            // Select and activate curse
            ActiveCurse = availableCurses[Random.Range(0, availableCurses.Count)];
            ApplyCurse(ActiveCurse);
            
            // Warning animation
            UIManager.Instance.ShowCurseWarning(ActiveCurse);
            yield return new WaitForSeconds(2f);
            
            // Curse period
            TimeRemaining = curseDuration;
            while (TimeRemaining > 0)
            {
                yield return new WaitForSeconds(1f);
                TimeRemaining--;
                UIManager.Instance.UpdateCurseTimer(TimeRemaining);
            }
            
            // Remove curse
            RemoveCurse(ActiveCurse);
        }
    }
    
    void ApplyCurse(CurseType curse)
    {
        Debug.Log($"Applying curse: {curse}");
        
        switch (curse)
        {
            case CurseType.DoubleSpawns:
                ARMonSpawner.Instance.spawnRateMultiplier = 2f;
                break;
                
            case CurseType.PowerfulMons:
                Mon.GlobalStatMultiplier = 1.5f;
                break;
                
            case CurseType.WeakenedPlayer:
                PlayerController.Instance.damageMultiplier = 0.5f;
                break;
                
            case CurseType.NoCapture:
                GoldenBook.CaptureDisabled = true;
                break;
                
            case CurseType.Invisibility:
                // Make random Mons invisible
                foreach (var mon in FindObjectsOfType<Mon>())
                {
                    if (Random.value < 0.5f)
                    {
                        mon.SetInvisible(true);
                    }
                }
                break;
                
            case CurseType.SpeedBoost:
                Time.timeScale = 1.5f; // Everything moves faster
                break;
                
            case CurseType.HealthDrain:
                StartCoroutine(HealthDrainCoroutine());
                break;
                
            case CurseType.ReverseControls:
                InputManager.Instance.ReverseControls = true;
                break;
                
            case CurseType.GiantMons:
                Mon.GlobalScaleMultiplier = 2f;
                break;
                
            case CurseType.SwarmMode:
                ARMonSpawner.Instance.maxMonsAtOnce = 20;
                break;
        }
        
        // Visual effects for curse
        CurseVisualEffects.Instance.ActivateEffect(curse);
    }
    
    void RemoveCurse(CurseType curse)
    {
        // Reset all modifications
        ARMonSpawner.Instance.spawnRateMultiplier = 1f;
        Mon.GlobalStatMultiplier = 1f;
        PlayerController.Instance.damageMultiplier = 1f;
        GoldenBook.CaptureDisabled = false;
        Time.timeScale = 1f;
        InputManager.Instance.ReverseControls = false;
        Mon.GlobalScaleMultiplier = 1f;
        ARMonSpawner.Instance.maxMonsAtOnce = 5;
        
        // Clear visual effects
        CurseVisualEffects.Instance.DeactivateAll();
        
        StopAllCoroutines();
        curseCoroutine = StartCoroutine(CurseCycle());
    }
    
    IEnumerator HealthDrainCoroutine()
    {
        while (ActiveCurse == CurseType.HealthDrain)
        {
            PlayerController.Instance.TakeDamage(1f);
            yield return new WaitForSeconds(2f);
        }
    }
}

public enum CurseType
{
    None,
    DoubleSpawns,
    PowerfulMons,
    WeakenedPlayer,
    NoCapture,
    Invisibility,
    SpeedBoost,
    HealthDrain,
    ReverseControls,
    GiantMons,
    SwarmMode
}
```

## Quick Testing Setup

### Test Scene Setup

1. Create a new scene "ARTestScene"
2. Add AR Session Origin
3. Add AR Session
4. Add AR Plane Manager
5. Add AR Raycast Manager
6. Set up basic UI Canvas
7. Add managers (CombatEngine, ARMonSpawner, CurseManager)

### Mock Mon for Testing

```csharp
// Create 5 test Mons quickly
public class TestMonGenerator : MonoBehaviour
{
    public GameObject basicMonPrefab;
    
    void Start()
    {
        CreateTestMons();
    }
    
    void CreateTestMons()
    {
        string[] names = {"Flameling", "Aquapal", "Grassie", "Rockster", "Ghosty"};
        MonType[] types = {MonType.Fire, MonType.Water, MonType.Grass, 
                          MonType.Rock, MonType.Ghost};
        
        for (int i = 0; i < 5; i++)
        {
            GameObject mon = Instantiate(basicMonPrefab);
            Mon monScript = mon.GetComponent<Mon>();
            
            monScript.monId = i + 1;
            monScript.monName = names[i];
            monScript.primaryType = types[i];
            monScript.maxHealth = 50 + (i * 10);
            monScript.attack = 10 + (i * 2);
            
            // Save as prefab variant
            string path = $"Assets/Prefabs/Mons/{names[i]}.prefab";
            PrefabUtility.SaveAsPrefabAsset(mon, path);
            
            DestroyImmediate(mon);
        }
    }
}
```

## Next Steps

1. **Backend API** - Set up Node.js server for multiplayer
2. **Mon Database** - Implement procedural generation
3. **Save System** - Local and cloud saves
4. **Tutorial** - Onboarding for new players
5. **Monetization** - Shop and IAP integration

This prototype provides the foundation for all core MonPrime mechanics!