# Aiming in Space: Unity Implementation

## Overview
Practical implementation of aiming and targeting systems in 3D space, including leading targets, ballistic trajectories, and intercept calculations.

## Learning Objectives
- Implement target leading calculations
- Create ballistic trajectory systems
- Handle 3D aiming mathematics
- Optimize for real-time performance

---

## Basic Target Leading

### Simple Leading System
```csharp
public class TargetLeading : MonoBehaviour
{
    public Transform target;
    public float projectileSpeed = 100f; // m/s
    public Transform aimPoint;
    
    void Update()
    {
        if (target == null) return;
        
        Vector3 leadPosition = CalculateLeadPosition();
        
        if (aimPoint != null)
        {
            aimPoint.position = leadPosition;
            transform.LookAt(leadPosition);
        }
    }
    
    Vector3 CalculateLeadPosition()
    {
        Vector3 targetPos = target.position;
        Vector3 targetVel = GetTargetVelocity();
        Vector3 shooterPos = transform.position;
        
        // Calculate time to intercept
        Vector3 relativePos = targetPos - shooterPos;
        Vector3 relativeVel = targetVel;
        
        // Solve quadratic: |relativePos + relativeVel * t| = projectileSpeed * t
        float a = Vector3.Dot(relativeVel, relativeVel) - projectileSpeed * projectileSpeed;
        float b = 2f * Vector3.Dot(relativePos, relativeVel);
        float c = Vector3.Dot(relativePos, relativePos);
        
        float discriminant = b * b - 4f * a * c;
        
        if (discriminant < 0)
        {
            // No solution, aim directly at target
            return targetPos;
        }
        
        float t1 = (-b + Mathf.Sqrt(discriminant)) / (2f * a);
        float t2 = (-b - Mathf.Sqrt(discriminant)) / (2f * a);
        
        // Use positive time (earliest intercept)
        float t = Mathf.Max(t1, t2);
        if (t < 0) t = Mathf.Max(t1, t2);
        
        // Calculate lead position
        return targetPos + targetVel * t;
    }
    
    Vector3 GetTargetVelocity()
    {
        // Get target's velocity (if it has Rigidbody)
        Rigidbody rb = target.GetComponent<Rigidbody>();
        if (rb != null)
            return rb.velocity;
        
        // Or estimate from position change
        return Vector3.zero; // Simplified
    }
}
```

---

## Ballistic Trajectory

### Ballistic Calculator
```csharp
public class BallisticTrajectory : MonoBehaviour
{
    public float gravity = 9.81f;
    public int trajectoryPoints = 50;
    public LineRenderer trajectoryLine;
    
    public Vector3 CalculateLaunchVelocity(Vector3 start, Vector3 target, float timeToTarget)
    {
        Vector3 displacement = target - start;
        
        // Solve: target = start + velocity * t + 0.5 * gravity * t^2
        // velocity = (target - start - 0.5 * gravity * t^2) / t
        Vector3 gravityTerm = Vector3.down * 0.5f * gravity * timeToTarget * timeToTarget;
        Vector3 velocity = (displacement - gravityTerm) / timeToTarget;
        
        return velocity;
    }
    
    public float CalculateTimeToTarget(Vector3 start, Vector3 target, float launchSpeed)
    {
        // Solve ballistic equation for time
        // This is simplified - would need to solve quadratic
        
        float horizontalDistance = Vector3.Distance(
            new Vector3(start.x, 0, start.z),
            new Vector3(target.x, 0, target.z)
        );
        
        // Approximate time (ignoring gravity for horizontal)
        return horizontalDistance / launchSpeed;
    }
    
    public List<Vector3> GenerateTrajectory(Vector3 start, Vector3 velocity, float duration)
    {
        List<Vector3> points = new List<Vector3>();
        
        for (int i = 0; i <= trajectoryPoints; i++)
        {
            float t = (float)i / trajectoryPoints * duration;
            Vector3 position = start + velocity * t + Vector3.down * 0.5f * gravity * t * t;
            points.Add(position);
        }
        
        return points;
    }
    
    public void DrawTrajectory(Vector3 start, Vector3 target, float launchSpeed)
    {
        float time = CalculateTimeToTarget(start, target, launchSpeed);
        Vector3 velocity = CalculateLaunchVelocity(start, target, time);
        List<Vector3> points = GenerateTrajectory(start, velocity, time);
        
        if (trajectoryLine != null)
        {
            trajectoryLine.positionCount = points.Count;
            trajectoryLine.SetPositions(points.ToArray());
        }
    }
}
```

---

## Intercept Calculation

### Intercept Solver
```csharp
public class InterceptCalculator : MonoBehaviour
{
    public bool CalculateIntercept(Vector3 shooterPos, Vector3 shooterVel,
                                   Vector3 targetPos, Vector3 targetVel,
                                   float projectileSpeed,
                                   out Vector3 interceptPoint, out float interceptTime)
    {
        Vector3 relativePos = targetPos - shooterPos;
        Vector3 relativeVel = targetVel - shooterVel;
        
        // Solve: |relativePos + relativeVel * t| = projectileSpeed * t
        float a = Vector3.Dot(relativeVel, relativeVel) - projectileSpeed * projectileSpeed;
        float b = 2f * Vector3.Dot(relativePos, relativeVel);
        float c = Vector3.Dot(relativePos, relativePos);
        
        float discriminant = b * b - 4f * a * c;
        
        if (discriminant < 0 || a == 0)
        {
            interceptPoint = targetPos;
            interceptTime = 0f;
            return false; // No solution
        }
        
        float sqrtDiscriminant = Mathf.Sqrt(discriminant);
        float t1 = (-b + sqrtDiscriminant) / (2f * a);
        float t2 = (-b - sqrtDiscriminant) / (2f * a);
        
        // Use positive time
        interceptTime = (t1 > 0) ? t1 : ((t2 > 0) ? t2 : 0f);
        
        if (interceptTime <= 0)
        {
            interceptPoint = targetPos;
            return false;
        }
        
        interceptPoint = targetPos + targetVel * interceptTime;
        return true;
    }
    
    public Vector3 CalculateAimDirection(Vector3 shooterPos, Vector3 interceptPoint, 
                                          float projectileSpeed)
    {
        Vector3 direction = interceptPoint - shooterPos;
        return direction.normalized;
    }
}
```

---

## Turret System

### Space Turret
```csharp
public class SpaceTurret : MonoBehaviour
{
    [Header("Turret Properties")]
    public Transform turretBase;
    public Transform turretBarrel;
    public float rotationSpeed = 90f; // degrees per second
    public float maxElevation = 80f;
    public float minElevation = -10f;
    
    [Header("Targeting")]
    public Transform target;
    public TargetLeading leadingSystem;
    public float trackingSpeed = 2f;
    
    [Header("Firing")]
    public GameObject projectilePrefab;
    public Transform firePoint;
    public float fireRate = 1f;
    private float nextFireTime = 0f;
    
    void Update()
    {
        if (target == null) return;
        
        // Calculate aim direction
        Vector3 aimDirection = CalculateAimDirection();
        
        // Rotate turret
        RotateTurret(aimDirection);
        
        // Fire if ready
        if (Time.time >= nextFireTime && IsOnTarget(aimDirection))
        {
            Fire();
            nextFireTime = Time.time + 1f / fireRate;
        }
    }
    
    Vector3 CalculateAimDirection()
    {
        if (leadingSystem != null)
        {
            Vector3 leadPos = leadingSystem.CalculateLeadPosition();
            return (leadPos - turretBarrel.position).normalized;
        }
        else
        {
            return (target.position - turretBarrel.position).normalized;
        }
    }
    
    void RotateTurret(Vector3 aimDirection)
    {
        // Calculate yaw (horizontal rotation)
        Vector3 horizontalDirection = new Vector3(aimDirection.x, 0, aimDirection.z).normalized;
        if (horizontalDirection.magnitude > 0.1f)
        {
            Quaternion targetYaw = Quaternion.LookRotation(horizontalDirection);
            turretBase.rotation = Quaternion.RotateTowards(
                turretBase.rotation, 
                targetYaw, 
                rotationSpeed * Time.deltaTime
            );
        }
        
        // Calculate pitch (elevation)
        float elevation = Mathf.Asin(aimDirection.y) * Mathf.Rad2Deg;
        elevation = Mathf.Clamp(elevation, minElevation, maxElevation);
        
        Quaternion targetPitch = Quaternion.Euler(-elevation, 0, 0);
        turretBarrel.localRotation = Quaternion.RotateTowards(
            turretBarrel.localRotation,
            targetPitch,
            rotationSpeed * Time.deltaTime
        );
    }
    
    bool IsOnTarget(Vector3 aimDirection)
    {
        Vector3 currentDirection = turretBarrel.forward;
        float angle = Vector3.Angle(currentDirection, aimDirection);
        return angle < 2f; // Within 2 degrees
    }
    
    void Fire()
    {
        if (projectilePrefab == null || firePoint == null) return;
        
        GameObject projectile = Instantiate(projectilePrefab, firePoint.position, firePoint.rotation);
        
        // Set projectile velocity
        Rigidbody rb = projectile.GetComponent<Rigidbody>();
        if (rb != null)
        {
            rb.velocity = firePoint.forward * leadingSystem.projectileSpeed;
        }
    }
}
```

---

## Missile Guidance

### Homing Missile
```csharp
public class HomingMissile : MonoBehaviour
{
    [Header("Missile Properties")]
    public Transform target;
    public float maxSpeed = 50f;
    public float acceleration = 20f;
    public float turnRate = 90f; // degrees per second
    public float proximityFuse = 10f;
    
    [Header("Guidance")]
    public float guidanceUpdateRate = 10f; // Hz
    private float lastGuidanceUpdate = 0f;
    private Vector3 currentVelocity;
    
    private Rigidbody rb;
    
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb == null)
            rb = gameObject.AddComponent<Rigidbody>();
        
        currentVelocity = transform.forward * maxSpeed;
    }
    
    void FixedUpdate()
    {
        if (target == null)
        {
            // Continue on last course
            rb.velocity = currentVelocity;
            return;
        }
        
        // Check proximity
        float distance = Vector3.Distance(transform.position, target.position);
        if (distance < proximityFuse)
        {
            Explode();
            return;
        }
        
        // Update guidance
        if (Time.time - lastGuidanceUpdate >= 1f / guidanceUpdateRate)
        {
            UpdateGuidance();
            lastGuidanceUpdate = Time.time;
        }
        
        // Apply steering
        Vector3 desiredDirection = (target.position - transform.position).normalized;
        Vector3 currentDirection = currentVelocity.normalized;
        
        // Calculate turn
        Vector3 cross = Vector3.Cross(currentDirection, desiredDirection);
        float angle = Vector3.Angle(currentDirection, desiredDirection);
        
        // Limit turn rate
        float maxTurn = turnRate * Time.fixedDeltaTime;
        angle = Mathf.Min(angle, maxTurn);
        
        if (angle > 0.1f)
        {
            Vector3 axis = cross.normalized;
            Quaternion rotation = Quaternion.AngleAxis(angle, axis);
            currentVelocity = rotation * currentVelocity;
        }
        else
        {
            currentVelocity = desiredDirection * currentVelocity.magnitude;
        }
        
        // Limit speed
        currentVelocity = Vector3.ClampMagnitude(currentVelocity, maxSpeed);
        
        // Apply velocity
        rb.velocity = currentVelocity;
        transform.rotation = Quaternion.LookRotation(currentVelocity);
    }
    
    void UpdateGuidance()
    {
        // Predict target position
        Rigidbody targetRb = target.GetComponent<Rigidbody>();
        if (targetRb != null)
        {
            // Use predicted intercept point
            InterceptCalculator calculator = new InterceptCalculator();
            Vector3 interceptPoint;
            float interceptTime;
            
            if (calculator.CalculateIntercept(
                transform.position, currentVelocity,
                target.position, targetRb.velocity,
                maxSpeed,
                out interceptPoint, out interceptTime))
            {
                // Aim for intercept point
                target = null; // Will be set to intercept point in actual implementation
            }
        }
    }
    
    void Explode()
    {
        // Explosion logic
        Destroy(gameObject);
    }
}
```

---

## Aiming UI

### Aiming Reticle
```csharp
public class AimingReticle : MonoBehaviour
{
    public RectTransform reticle;
    public Camera playerCamera;
    public float reticleSize = 50f;
    
    public Transform target;
    public TargetLeading leadingSystem;
    
    void Update()
    {
        if (target == null || leadingSystem == null) return;
        
        Vector3 worldPosition = leadingSystem.CalculateLeadPosition();
        Vector3 screenPosition = playerCamera.WorldToScreenPoint(worldPosition);
        
        // Check if on screen
        if (screenPosition.z > 0 && 
            screenPosition.x > 0 && screenPosition.x < Screen.width &&
            screenPosition.y > 0 && screenPosition.y < Screen.height)
        {
            reticle.gameObject.SetActive(true);
            reticle.position = screenPosition;
        }
        else
        {
            reticle.gameObject.SetActive(false);
        }
    }
}
```

---

## Exercises for Self-Learning

1. **Target Leading**: Implement basic target leading system
2. **Ballistic Trajectory**: Create ballistic trajectory calculator
3. **Intercept Calculation**: Implement intercept solver
4. **Turret System**: Create automated turret with aiming
5. **Missile Guidance**: Implement homing missile system

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)

---

## References
- Ballistic trajectory calculations
- 3D vector mathematics
- Target prediction algorithms
- Missile guidance systems
