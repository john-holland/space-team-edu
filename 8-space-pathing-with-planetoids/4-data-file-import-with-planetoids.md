# Data File Import with Planetoids

## Overview
Importing and loading data files containing planetoid information, including orbital data, physical properties, and spatial relationships for space pathfinding systems.

## Learning Objectives
- Understand planetoid data formats
- Learn how to import orbital data
- Master coordinate system conversions
- Handle large-scale data efficiently

---

## Planetoid Data Formats

### Common Formats

#### CSV Format
Simple comma-separated values:
```csv
name,mass,radius,semiMajorAxis,eccentricity,inclination,longitudeOfAscendingNode,argumentOfPeriapsis,meanAnomaly
Earth,5.972e24,6371000,1.496e11,0.0167,0,0,102.9,357.5
Mars,6.39e23,3389500,2.279e11,0.0934,1.85,49.6,286.5,19.4
```

#### JSON Format
Structured data:
```json
{
  "planetoids": [
    {
      "name": "Earth",
      "mass": 5.972e24,
      "radius": 6371000,
      "orbitalElements": {
        "semiMajorAxis": 1.496e11,
        "eccentricity": 0.0167,
        "inclination": 0,
        "longitudeOfAscendingNode": 0,
        "argumentOfPeriapsis": 102.9,
        "meanAnomaly": 357.5
      },
      "physicalProperties": {
        "surfaceGravity": 9.81,
        "atmosphere": true
      }
    }
  ]
}
```

#### Binary Format
Custom binary format for efficiency:
```
[Header]
    Magic number
    Version
    Planetoid count
    
[Planetoid 1]
    Name (string)
    Mass (double)
    Radius (double)
    Orbital elements (6 doubles)
    Physical properties
    
[Planetoid 2]
    ...
```

---

## Orbital Elements Import

### Classical Orbital Elements
Six parameters define an orbit:

1. **Semi-major axis** $a$: Size of orbit
2. **Eccentricity** $e$: Shape (0 = circle, <1 = ellipse)
3. **Inclination** $i$: Tilt of orbit plane
4. **Longitude of ascending node** $\Omega$: Orientation
5. **Argument of periapsis** $\omega$: Ellipse orientation
6. **Mean anomaly** $M$ or **True anomaly** $\nu$: Position

### Importing Orbital Elements
```csharp
public class OrbitalElementImporter
{
    public CelestialBody ImportFromOrbitalElements(string name, OrbitalElements elements, 
                                                   CelestialBody centralBody)
    {
        CelestialBody body = new GameObject(name).AddComponent<CelestialBody>();
        
        // Convert orbital elements to position and velocity
        Vector3 position = OrbitalElementsToPosition(elements, centralBody);
        Vector3 velocity = OrbitalElementsToVelocity(elements, centralBody);
        
        body.transform.position = position;
        body.initialVelocity = velocity;
        body.centralBody = centralBody;
        
        return body;
    }
    
    Vector3 OrbitalElementsToPosition(OrbitalElements elements, CelestialBody centralBody)
    {
        // Calculate true anomaly from mean anomaly
        float trueAnomaly = MeanToTrueAnomaly(elements.meanAnomaly, elements.eccentricity);
        
        // Calculate position in perifocal frame
        float r = elements.semiMajorAxis * (1 - elements.eccentricity * elements.eccentricity) /
                  (1 + elements.eccentricity * Mathf.Cos(trueAnomaly));
        
        float x = r * Mathf.Cos(trueAnomaly);
        float y = r * Mathf.Sin(trueAnomaly);
        float z = 0;
        
        Vector3 perifocalPos = new Vector3(x, y, z);
        
        // Rotate to inertial frame
        Quaternion rotation = CalculateRotationMatrix(elements);
        Vector3 inertialPos = rotation * perifocalPos;
        
        return centralBody.transform.position + inertialPos;
    }
    
    Vector3 OrbitalElementsToVelocity(OrbitalElements elements, CelestialBody centralBody)
    {
        // Calculate velocity from orbital elements
        float G = 6.674e-11f;
        float mu = G * centralBody.mass;
        float a = elements.semiMajorAxis;
        float e = elements.eccentricity;
        
        float trueAnomaly = MeanToTrueAnomaly(elements.meanAnomaly, e);
        float r = a * (1 - e * e) / (1 + e * Mathf.Cos(trueAnomaly));
        
        float v = Mathf.Sqrt(mu * (2f / r - 1f / a));
        float flightPathAngle = Mathf.Atan((e * Mathf.Sin(trueAnomaly)) / 
                                          (1 + e * Mathf.Cos(trueAnomaly)));
        
        // Calculate velocity components
        float vx = v * Mathf.Sin(flightPathAngle);
        float vy = v * Mathf.Cos(flightPathAngle);
        
        Vector3 perifocalVel = new Vector3(vx, vy, 0);
        
        // Rotate to inertial frame
        Quaternion rotation = CalculateRotationMatrix(elements);
        return rotation * perifocalVel;
    }
    
    float MeanToTrueAnomaly(float meanAnomaly, float eccentricity)
    {
        // Solve Kepler's equation: M = E - e*sin(E)
        // Use iterative method
        float E = meanAnomaly; // Initial guess
        for (int i = 0; i < 10; i++)
        {
            E = meanAnomaly + eccentricity * Mathf.Sin(E);
        }
        
        // Convert eccentric anomaly to true anomaly
        float trueAnomaly = 2f * Mathf.Atan(
            Mathf.Sqrt((1 + eccentricity) / (1 - eccentricity)) * Mathf.Tan(E / 2f)
        );
        
        return trueAnomaly;
    }
    
    Quaternion CalculateRotationMatrix(OrbitalElements elements)
    {
        // Rotation from perifocal to inertial frame
        // Combines: rotation around z (Ω), rotation around x (i), rotation around z (ω)
        Quaternion rot1 = Quaternion.Euler(0, 0, elements.longitudeOfAscendingNode);
        Quaternion rot2 = Quaternion.Euler(elements.inclination, 0, 0);
        Quaternion rot3 = Quaternion.Euler(0, 0, elements.argumentOfPeriapsis);
        
        return rot1 * rot2 * rot3;
    }
}

[System.Serializable]
public class OrbitalElements
{
    public float semiMajorAxis;
    public float eccentricity;
    public float inclination;
    public float longitudeOfAscendingNode;
    public float argumentOfPeriapsis;
    public float meanAnomaly;
}
```

---

## CSV Import

### CSV Parser
```csharp
public class PlanetoidCSVImporter
{
    public List<PlanetoidData> ImportFromCSV(string filePath)
    {
        List<PlanetoidData> planetoids = new List<PlanetoidData>();
        
        string[] lines = File.ReadAllLines(filePath);
        
        // Skip header line
        for (int i = 1; i < lines.Length; i++)
        {
            string[] values = lines[i].Split(',');
            
            if (values.Length < 9) continue;
            
            PlanetoidData data = new PlanetoidData
            {
                name = values[0].Trim(),
                mass = float.Parse(values[1]),
                radius = float.Parse(values[2]),
                orbitalElements = new OrbitalElements
                {
                    semiMajorAxis = float.Parse(values[3]),
                    eccentricity = float.Parse(values[4]),
                    inclination = float.Parse(values[5]),
                    longitudeOfAscendingNode = float.Parse(values[6]),
                    argumentOfPeriapsis = float.Parse(values[7]),
                    meanAnomaly = float.Parse(values[8])
                }
            };
            
            planetoids.Add(data);
        }
        
        return planetoids;
    }
}

[System.Serializable]
public class PlanetoidData
{
    public string name;
    public float mass;
    public float radius;
    public OrbitalElements orbitalElements;
    public PhysicalProperties physicalProperties;
}

[System.Serializable]
public class PhysicalProperties
{
    public float surfaceGravity;
    public bool hasAtmosphere;
    public float atmosphereHeight;
    public float surfacePressure;
}
```

---

## JSON Import

### JSON Parser
```csharp
public class PlanetoidJSONImporter
{
    [System.Serializable]
    public class PlanetoidDataList
    {
        public PlanetoidData[] planetoids;
    }
    
    public List<PlanetoidData> ImportFromJSON(string filePath)
    {
        string json = File.ReadAllText(filePath);
        PlanetoidDataList dataList = JsonUtility.FromJson<PlanetoidDataList>(json);
        
        return new List<PlanetoidData>(dataList.planetoids);
    }
}
```

---

## Binary Format Import

### Binary Reader
```csharp
public class PlanetoidBinaryImporter
{
    public List<PlanetoidData> ImportFromBinary(string filePath)
    {
        List<PlanetoidData> planetoids = new List<PlanetoidData>();
        
        using (BinaryReader reader = new BinaryReader(File.OpenRead(filePath)))
        {
            // Read header
            uint magic = reader.ReadUInt32();
            if (magic != 0x504C414E) // "PLAN"
            {
                Debug.LogError("Invalid planetoid file!");
                return null;
            }
            
            ushort version = reader.ReadUInt16();
            int count = reader.ReadInt32();
            
            // Read planetoids
            for (int i = 0; i < count; i++)
            {
                PlanetoidData data = ReadPlanetoid(reader);
                planetoids.Add(data);
            }
        }
        
        return planetoids;
    }
    
    PlanetoidData ReadPlanetoid(BinaryReader reader)
    {
        PlanetoidData data = new PlanetoidData();
        
        // Read name
        int nameLength = reader.ReadInt32();
        byte[] nameBytes = reader.ReadBytes(nameLength);
        data.name = System.Text.Encoding.UTF8.GetString(nameBytes);
        
        // Read physical properties
        data.mass = reader.ReadDouble();
        data.radius = reader.ReadDouble();
        
        // Read orbital elements
        data.orbitalElements = new OrbitalElements
        {
            semiMajorAxis = reader.ReadDouble(),
            eccentricity = reader.ReadDouble(),
            inclination = reader.ReadDouble(),
            longitudeOfAscendingNode = reader.ReadDouble(),
            argumentOfPeriapsis = reader.ReadDouble(),
            meanAnomaly = reader.ReadDouble()
        };
        
        return data;
    }
}
```

---

## Coordinate System Conversion

### Converting Imported Data
```csharp
public class CoordinateConverter
{
    public void ConvertToUnityCoordinates(List<PlanetoidData> planetoids, 
                                        float scaleFactor = 1f)
    {
        foreach (var planetoid in planetoids)
        {
            // Scale distances
            planetoid.orbitalElements.semiMajorAxis *= scaleFactor;
            planetoid.radius *= scaleFactor;
            
            // Convert angles (degrees to radians if needed)
            planetoid.orbitalElements.inclination *= Mathf.Deg2Rad;
            planetoid.orbitalElements.longitudeOfAscendingNode *= Mathf.Deg2Rad;
            planetoid.orbitalElements.argumentOfPeriapsis *= Mathf.Deg2Rad;
            planetoid.orbitalElements.meanAnomaly *= Mathf.Deg2Rad;
        }
    }
}
```

---

## Data Validation

### Validation Checks
```csharp
public class PlanetoidDataValidator
{
    public bool Validate(PlanetoidData data)
    {
        // Check mass
        if (data.mass <= 0)
        {
            Debug.LogError($"{data.name}: Invalid mass");
            return false;
        }
        
        // Check radius
        if (data.radius <= 0)
        {
            Debug.LogError($"{data.name}: Invalid radius");
            return false;
        }
        
        // Check orbital elements
        if (data.orbitalElements.semiMajorAxis <= 0)
        {
            Debug.LogError($"{data.name}: Invalid semi-major axis");
            return false;
        }
        
        if (data.orbitalElements.eccentricity < 0 || data.orbitalElements.eccentricity >= 1)
        {
            Debug.LogError($"{data.name}: Invalid eccentricity");
            return false;
        }
        
        // Check angles
        if (data.orbitalElements.inclination < 0 || data.orbitalElements.inclination > Mathf.PI)
        {
            Debug.LogError($"{data.name}: Invalid inclination");
            return false;
        }
        
        return true;
    }
}
```

---

## Performance Optimization

### Streaming Import
For large files, import incrementally:

```csharp
public IEnumerator ImportPlanetoidsAsync(string filePath, 
                                        System.Action<List<PlanetoidData>> callback)
{
    List<PlanetoidData> planetoids = new List<PlanetoidData>();
    
    // Read file in chunks
    using (StreamReader reader = new StreamReader(filePath))
    {
        string line;
        int lineCount = 0;
        
        while ((line = reader.ReadLine()) != null)
        {
            if (lineCount++ == 0) continue; // Skip header
            
            PlanetoidData data = ParseLine(line);
            if (data != null)
                planetoids.Add(data);
            
            // Yield every 100 lines
            if (lineCount % 100 == 0)
                yield return null;
        }
    }
    
    callback?.Invoke(planetoids);
}
```

---

## Exercises for Self-Learning

1. **CSV Import**: Implement CSV planetoid importer
2. **JSON Import**: Create JSON import system
3. **Orbital Conversion**: Convert orbital elements to positions
4. **Validation**: Add data validation checks
5. **Async Loading**: Implement async import for large files

---

## Next Steps
- **Usage**: Review use cases (1-usage.md)
- **Math**: Understand mathematics (2-math.md)
- **Implementation**: See Unity code (3-unity-implementation.md)

---

## References
- Orbital mechanics data formats
- File I/O in Unity
- Coordinate system conversions
- Data validation techniques
