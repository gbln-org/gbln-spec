# GBLN Examples
## Comprehensive Usage Guide

**Version**: 1.0  
**Date**: 2025-01-21  
**Authors**: Vivian Voss

---

## Table of Contents

1. [Basic Examples](#basic-examples)
2. [Configuration Files](#configuration-files)
3. [API Responses](#api-responses)
4. [Database Exports](#database-exports)
5. [IoT Device Data](#iot-device-data)
6. [Mobile App Data](#mobile-app-data)
7. [Complex Nested Structures](#complex-nested-structures)
8. [I/O Format Workflow](#io-format-workflow)
9. [Edge Cases](#edge-cases)
10. [Best Practises](#best-practises)
11. [Anti-Patterns](#anti-patterns)

---

## Basic Examples

### Simple Primitives

```gbln
:| Integers
age<i8>(25)
user_id<u32>(12345)
timestamp<i64>(1737475200)

:| Floats
price<f32>(19.99)
latitude<f64>(51.5074)
scientific<f64>(1.23e-10)

:| Strings
name<s32>(Alice Johnson)
email<s64>(alice@example.com)
country<s2>(DE)

:| Boolean
active<b>(t)
verified<b>(true)
debug<b>(0)

:| Null
optional<n>()
missing<n>(null)
```

### Simple Object

```gbln
user{
    id<u32>(12345)
    name<s64>(Alice Johnson)
    age<i8>(25)
    email<s64>(alice@example.com)
    active<b>(t)
}
```

### Simple Arrays

```gbln
:| Homogeneous array - all same type
numbers<i32>[1 2 3 4 5]
tags<s16>[rust python golang javascript]
scores<f32>[98.5 87.2 92.1]

:| Mixed array - different types
mixed[
    <i32>(42)
    <s64>(hello world)
    <b>(t)
    <f32>(3.14)
]

:| Object array
users[
    {id<u32>(1) name<s32>(Alice)}
    {id<u32>(2) name<s32>(Bob)}
    {id<u32>(3) name<s32>(Charlie)}
]
```

---

## Configuration Files

### Application Configuration

```gbln
:| Application Configuration - Production Environment
app{
    :| Basic settings
    name<s32>(My Application)
    version<s16>(1.2.3)
    environment<s8>(prod)
    
    :| Server configuration
    server{
        host<s64>(app.example.com)
        port<u16>(8080)
        workers<u8>(4)
        timeout_ms<u32>(30000)
        keep_alive<b>(t)
    }
    
    :| Database connection
    database{
        driver<s16>(postgresql)
        host<s64>(db.example.com)
        port<u16>(5432)
        name<s32>(app_production)
        username<s32>(app_user)
        max_connections<u8>(20)
        ssl<b>(t)
    }
    
    :| Feature flags
    features{
        analytics<b>(t)
        new_ui<b>(f)
        beta_features<b>(f)
    }
    
    :| Logging
    logging{
        level<s8>(info)
        format<s16>(json)
        output<s32>(stdout)
    }
}
```

### Web Server Configuration

```gbln
:| Nginx-style Web Server Configuration
server{
    :| Server identification
    name<s64>(example.com)
    aliases<s64>[www.example.com api.example.com]
    
    :| Network settings
    listen<u16>(443)
    protocol<s8>(https)
    
    :| SSL/TLS
    ssl{
        certificate<s128>(/etc/ssl/certs/example.com.crt)
        certificate_key<s128>(/etc/ssl/private/example.com.key)
        protocols<s8>[TLSv1.2 TLSv1.3]
    }
    
    :| Locations
    locations[
        {
            path<s32>(/)
            root<s64>(/var/www/html)
            index<s32>(index.html)
        }
        {
            path<s32>(/api)
            proxy_pass<s64>(http://localhost:3000)
            timeout<u16>(60)
        }
    ]
    
    :| Limits
    max_body_size<u32>(10485760)      :| 10MB in bytes
    rate_limit<u16>(100)               :| Requests per second
}
```

### Build Configuration

```gbln
:| Rust Project Configuration (Cargo.toml equivalent)
package{
    name<s32>(gbln)
    version<s16>(0.1.0)
    authors<s64>[Vivian Voss <ask+gbln@vvoss.dev>]
    edition<s8>(2021)
    licence<s8>(MIT)
    description<s256>(GBLN parser and serialisation library)
}

dependencies{
    :| Production dependencies
    criterion<s16>(0.5)
    
    :| Dev dependencies marked with dev flag
    proptest{
        version<s8>(1.0)
        dev<b>(t)
    }
}

profile_release{
    opt_level<u8>(3)
    lto<b>(t)
    codegen_units<u8>(1)
    strip<b>(t)
}
```

---

## API Responses

### REST API Success Response

```gbln
:| GET /api/users/12345 - Success Response
response{
    :| HTTP metadata
    status<u16>(200)
    status_text<s16>(OK)
    
    :| Response data
    data{
        user{
            id<u32>(12345)
            username<s16>(alice_dev)
            email<s64>(alice@example.com)
            full_name<s64>(Alice Johnson)
            avatar_url<s128>(https://cdn.example.com/avatars/12345.jpg)
            
            :| Account status
            created_at<u64>(1609459200)
            last_login<u64>(1737475200)
            verified<b>(t)
            active<b>(t)
            
            :| User metadata
            metadata{
                role<s16>(admin)
                department<s32>(Engineering)
                location<s32>(San Francisco)
            }
            
            :| Permissions
            permissions<s32>[
                read_users
                write_users
                read_reports
                write_reports
                admin_access
            ]
        }
    }
    
    :| Request metadata
    meta{
        request_id<s64>(req-2025-01-21-abc123)
        timestamp<u64>(1737475200)
        duration_ms<u16>(45)
        api_version<s8>(v1)
    }
}
```

### REST API Error Response

```gbln
:| POST /api/users - Validation Error Response
response{
    :| HTTP metadata
    status<u16>(400)
    status_text<s32>(Bad Request)
    
    :| Error details
    error{
        code<s32>(VALIDATION_ERROR)
        message<s128>(Request validation failed)
        
        :| Field-level errors
        errors[
            {
                field<s32>(email)
                code<s32>(INVALID_FORMAT)
                message<s64>(Email address is invalid)
            }
            {
                field<s32>(age)
                code<s32>(OUT_OF_RANGE)
                message<s64>(Age must be between 0 and 120)
            }
        ]
    }
    
    :| Request metadata
    meta{
        request_id<s64>(req-2025-01-21-xyz789)
        timestamp<u64>(1737475200)
        api_version<s8>(v1)
    }
}
```

### GraphQL-Style Response

```gbln
:| GraphQL Query Response
response{
    data{
        user{
            id<u32>(12345)
            name<s64>(Alice Johnson)
            
            posts[
                {
                    id<u32>(1)
                    title<s128>(Introduction to GBLN)
                    excerpt<s256>(Learn about type-safe serialisation...)
                    published_at<u64>(1737388800)
                    likes<u32>(142)
                }
                {
                    id<u32>(2)
                    title<s128>(Advanced GBLN Patterns)
                    excerpt<s256>(Explore complex data structures...)
                    published_at<u64>(1737475200)
                    likes<u32>(87)
                }
            ]
            
            followers{
                total_count<u32>(1523)
                edges[
                    {
                        node{
                            id<u32>(67890)
                            name<s32>(Bob Smith)
                        }
                    }
                ]
            }
        }
    }
}
```

---

## Database Exports

### User Table Export

```gbln
:| Users Table - Database Export
users[
    {
        id<u32>(1)
        username<s16>(alice_dev)
        email<s64>(alice@example.com)
        full_name<s64>(Alice Johnson)
        age<i8>(25)
        created_at<u64>(1609459200)
        updated_at<u64>(1737475200)
        active<b>(t)
        verified<b>(t)
        role<s16>(admin)
    }
    {
        id<u32>(2)
        username<s16>(bob_designer)
        email<s64>(bob@example.com)
        full_name<s64>(Bob Smith)
        age<i8>(30)
        created_at<u64>(1612137600)
        updated_at<u64>(1737475200)
        active<b>(t)
        verified<b>(t)
        role<s16>(user)
    }
    {
        id<u32>(3)
        username<s16>(charlie_pm)
        email<s64>(charlie@example.com)
        full_name<s64>(Charlie Brown)
        age<i8>(35)
        created_at<u64>(1614556800)
        updated_at<u64>(1737389000)
        active<b>(f)
        verified<b>(t)
        role<s16>(user)
    }
]
```

### Relational Data with Foreign Keys

```gbln
:| Blog Posts with Authors and Comments
posts[
    {
        id<u32>(1)
        author_id<u32>(12345)
        title<s128>(Introduction to GBLN)
        content<s1024>(GBLN is a type-safe serialisation format...)
        published_at<u64>(1737388800)
        
        comments[
            {
                id<u32>(101)
                user_id<u32>(67890)
                text<s512>(Great introduction! Very clear.)
                created_at<u64>(1737400000)
            }
            {
                id<u32>(102)
                user_id<u32>(11111)
                text<s512>(Looking forward to trying this!)
                created_at<u64>(1737410000)
            }
        ]
        
        tags<s16>[programming serialisation rust]
    }
]
```

---

## IoT Device Data

### Environmental Sensor

```gbln
:| Environmental Sensor Reading
sensor{
    :| Device identification
    device_id<s16>(SENS-ENV-001)
    location<s64>(Building A, Floor 3, Room 305)
    firmware_version<s16>(2.1.4)
    
    :| Current readings (all within acceptable ranges)
    readings{
        temperature<f32>(22.5)        :| Celsius
        humidity<u8>(65)               :| Percentage (0-100)
        pressure<f32>(1013.25)        :| hPa
        air_quality_index<u16>(85)    :| 0-500 scale
        co2_ppm<u16>(420)             :| Parts per million
        light_lux<u16>(450)           :| Lux units
    }
    
    :| Historical data (last 10 readings)
    temperature_history<f32>[
        22.1 22.3 22.5 22.4 22.6
        22.5 22.7 22.5 22.3 22.4
    ]
    
    :| Device status
    status{
        battery_percentage<u8>(87)
        signal_strength<i8>(-45)      :| dBm
        last_calibration<u64>(1735689600)
        uptime_seconds<u32>(2592000)  :| 30 days
        errors<u16>(0)
        active<b>(t)
    }
}
```

### Smart Home Device

```gbln
:| Smart Thermostat Configuration
thermostat{
    device_id<s16>(THERM-001)
    room<s32>(Living Room)
    
    :| Current state
    current{
        temperature<f32>(21.5)
        target_temperature<f32>(22.0)
        mode<s8>(heat)
        fan_speed<s8>(auto)
        power<b>(t)
    }
    
    :| Schedule
    schedule[
        {
            day<s8>(monday)
            time<s8>(06:00)
            temperature<f32>(22.0)
        }
        {
            day<s8>(monday)
            time<s8>(22:00)
            temperature<f32>(18.0)
        }
    ]
    
    :| Energy usage
    energy{
        today_kwh<f32>(3.45)
        this_month_kwh<f32>(95.2)
        cost_today<f32>(0.52)         :| Currency units
    }
}
```

---

## Mobile App Data

### User Preferences

```gbln
:| Mobile App User Preferences
preferences{
    :| Appearance
    appearance{
        theme<s8>(dark)
        accent_colour<s16>(blue)
        font_size<u8>(14)              :| Points
        high_contrast<b>(f)
    }
    
    :| Notifications
    notifications{
        enabled<b>(t)
        sound<b>(t)
        vibration<b>(t)
        badge<b>(t)
        
        channels{
            messages<b>(t)
            updates<b>(t)
            marketing<b>(f)
        }
    }
    
    :| Privacy
    privacy{
        analytics<b>(t)
        crash_reports<b>(t)
        personalised_ads<b>(f)
        location_tracking<b>(f)
    }
    
    :| Language and region
    localisation{
        language<s2>(en)
        region<s2>(GB)
        timezone<s32>(Europe/London)
        date_format<s16>(DD/MM/YYYY)
        time_format<s8>(24h)
    }
}
```

### Offline Data Cache

```gbln
:| Mobile App Offline Cache
cache{
    version<u16>(1)
    last_sync<u64>(1737475200)
    
    :| Cached articles
    articles[
        {
            id<u32>(101)
            title<s128>(Getting Started with GBLN)
            author<s32>(Alice Johnson)
            read<b>(t)
            bookmarked<b>(t)
            cached_at<u64>(1737400000)
        }
        {
            id<u32>(102)
            title<s128>(Advanced Data Patterns)
            author<s32>(Bob Smith)
            read<b>(f)
            bookmarked<b>(t)
            cached_at<u64>(1737450000)
        }
    ]
    
    :| Pending sync operations
    pending_sync[
        {
            operation<s8>(update)
            resource<s16>(articles)
            id<u32>(101)
            data{read<b>(t)}
        }
    ]
}
```

---

## Complex Nested Structures

### E-Commerce Product

```gbln
:| E-Commerce Product Listing
product{
    :| Basic information
    id<u32>(67890)
    sku<s16>(PROD-MOUSE-001)
    name<s64>(Ergonomic Wireless Mouse)
    brand<s32>(TechGear)
    category<s32>(Electronics)
    
    :| Detailed description
    description{
        short<s128>(Comfortable wireless mouse with precision tracking)
        long<s1024>(Our ergonomic wireless mouse features advanced precision tracking, customisable buttons, and extended battery life. Perfect for both office work and gaming. The contoured design reduces hand fatigue during extended use.)
        
        features<s128>[
            Wireless connectivity
            Ergonomic design
            Precision optical sensor
            Customisable buttons
            Long battery life
        ]
    }
    
    :| Pricing
    pricing{
        base_price<f32>(29.99)
        currency<s3>(USD)
        discount_percentage<f32>(15.0)
        final_price<f32>(25.49)
        tax_inclusive<b>(f)
    }
    
    :| Inventory
    inventory{
        stock<u16>(150)
        warehouse<s32>(WH-CENTRAL-01)
        reserved<u16>(5)
        available<u16>(145)
        restock_date<u64>(1738080000)
    }
    
    :| Technical specifications
    specifications{
        dimensions{
            length_mm<u16>(120)
            width_mm<u8>(60)
            height_mm<u8>(40)
            weight_g<u16>(85)
        }
        
        connectivity{
            type<s16>(wireless)
            protocol<s8>(RF)
            range_metres<u8>(10)
        }
        
        power{
            battery_type<s8>(AAA)
            battery_count<u8>(2)
            battery_life_hours<u16>(720)    :| 30 days
        }
        
        compatibility<s16>[Windows macOS Linux]
    }
    
    :| Media
    images<s128>[
        https://cdn.example.com/products/mouse-main.jpg
        https://cdn.example.com/products/mouse-side.jpg
        https://cdn.example.com/products/mouse-top.jpg
    ]
    
    :| Customer ratings
    ratings{
        average<f32>(4.7)
        count<u32>(2543)
        
        distribution{
            five_star<u32>(1823)
            four_star<u32>(512)
            three_star<u32>(156)
            two_star<u32>(42)
            one_star<u32>(10)
        }
    }
    
    :| Metadata
    created_at<u64>(1704067200)
    updated_at<u64>(1737475200)
    published<b>(t)
    featured<b>(f)
}
```

### Company Organisational Structure

```gbln
:| Company Organisational Structure
company{
    name<s64>(TechCorp International)
    founded<u16>(2010)
    headquarters<s32>(San Francisco, CA)
    
    :| Leadership
    leadership{
        ceo{
            name<s64>(Alice Johnson)
            employee_id<u32>(1)
            email<s64>(alice.johnson@techcorp.com)
            joined<u64>(1262304000)
        }
        
        cto{
            name<s64>(Bob Smith)
            employee_id<u32>(2)
            email<s64>(bob.smith@techcorp.com)
            joined<u64>(1262304000)
        }
    }
    
    :| Departments
    departments[
        {
            name<s32>(Engineering)
            head<s64>(Charlie Brown)
            employee_count<u16>(85)
            budget<u32>(15000000)
            
            teams[
                {
                    name<s32>(Backend)
                    lead<s32>(Dave Wilson)
                    members<u8>(25)
                    technologies<s16>[Rust Python Go]
                }
                {
                    name<s32>(Frontend)
                    lead<s32>(Eve Davis)
                    members<u8>(20)
                    technologies<s16>[TypeScript React Vue]
                }
                {
                    name<s32>(DevOps)
                    lead<s32>(Frank Miller)
                    members<u8>(15)
                    technologies<s16>[Kubernetes Docker Terraform]
                }
            ]
        }
        {
            name<s32>(Product)
            head<s64>(Grace Lee)
            employee_count<u16>(30)
            budget<u32>(5000000)
            
            teams[
                {
                    name<s32>(Product Management)
                    lead<s32>(Henry Taylor)
                    members<u8>(12)
                }
                {
                    name<s32>(Design)
                    lead<s32>(Iris Anderson)
                    members<u8>(18)
                }
            ]
        }
    ]
    
    :| Locations
    offices[
        {
            city<s32>(San Francisco)
            country<s2>(US)
            employees<u16>(150)
            address<s128>(123 Market Street, San Francisco, CA 94102)
        }
        {
            city<s32>(London)
            country<s2>(GB)
            employees<u16>(75)
            address<s128>(456 Oxford Street, London, W1D 1BS)
        }
        {
            city<s32>(Berlin)
            country<s2>(DE)
            employees<u16>(50)
            address<s128>(789 Unter den Linden, 10117 Berlin)
        }
    ]
}
```

---

## I/O Format Workflow

### Overview

GBLN uses a dual-file system: human-editable source files (`.gbln`) and optimised I/O files (`.io.gbln.xz`).

### Example 1: Configuration File Workflow

**Step 1: Create Human-Editable Source**

`config.gbln`:
```gbln
:| Application Configuration
:| Updated: 2025-01-24
app{
  name<s32>(My Application)
  version<s16>(1.2.3)
  
  server{
    host<s64>(api.example.com)
    port<u16>(8080)
    workers<u8>(4)
    timeout_ms<u32>(30000)
  }
  
  database{
    host<s64>(db.example.com)
    port<u16>(5432)
    name<s32>(production_db)
  }
}
```

**Step 2: Generate I/O Format**

```bash
# Generate optimised I/O format (MINI GBLN + XZ compressed)
gbln write config.gbln

# Output: config.io.gbln.xz created
# File size: config.gbln (287 bytes) → config.io.gbln.xz (68 bytes, -76%)
```

**Step 3: Application Uses I/O Format**

```rust
// Application code
use gbln::read_io;

let value = read_io("config.io.gbln.xz")?;
// Automatically decompresses and parses

let port = value["app"]["server"]["port"].as_u16().unwrap();
println!("Server port: {}", port);  // 8080
```

**Step 4: Update Source from I/O File (if modified at runtime)**

```bash
# Application modified config.io.gbln.xz at runtime
# Update human-readable source to see changes
gbln read config.gbln

# Output: config.gbln updated from config.io.gbln.xz
```

---

### Example 2: API Response Workflow

**Server-Side: Generate I/O Format**

```rust
// Server code
use gbln::{Value, write_io, GblnConfig};
use std::path::Path;

// Create response data
let mut user = HashMap::new();
user.insert("id", Value::U32(12345));
user.insert("name", Value::Str("Alice".into()));
user.insert("email", Value::Str("alice@example.com".into()));

let response = Value::Object(user);

// Write to I/O format (MINI + XZ)
let config = GblnConfig::default();  // mini_mode=true, compress=true
write_io(&response, Path::new("response.io.gbln.xz"), &config)?;

// Send compressed response to client
send_file("response.io.gbln.xz");  // 95% less bandwidth than JSON
```

**Client-Side: Parse I/O Format**

```python
# Client code (Python)
import gbln

# Receive and parse I/O format
response = gbln.read_io("response.io.gbln.xz")
# Automatically decompresses and parses

user_id = response["id"]  # 12345
user_name = response["name"]  # "Alice"
```

---

### Example 3: LLM Context Optimization

**Prepare Data for LLM Context**

```bash
# Start with human-readable configs
ls configs/
# → server.gbln, database.gbln, cache.gbln

# Generate I/O formats
for file in configs/*.gbln; do
  gbln write "$file"
done

# Result: server.io.gbln.xz, database.io.gbln.xz, cache.io.gbln.xz
# Size: 1,200 bytes total (vs 4,800 bytes for .gbln files)
```

**Use in LLM Prompt**

```python
import gbln
import openai

# Load I/O formats (decompressed for LLM)
server_cfg = gbln.read_io("configs/server.io.gbln.xz")
db_cfg = gbln.read_io("configs/database.io.gbln.xz")
cache_cfg = gbln.read_io("configs/cache.io.gbln.xz")

# Convert to MINI GBLN strings for LLM context
server_str = gbln.to_string(server_cfg, mini=True)
db_str = gbln.to_string(db_cfg, mini=True)
cache_str = gbln.to_string(cache_cfg, mini=True)

# Create prompt with minimal token usage
prompt = f"""
Current configurations:
Server: {server_str}
Database: {db_str}
Cache: {cache_str}

Suggest optimizations for high-traffic scenario...
"""

# 84% fewer tokens than equivalent JSON
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}]
)
```

---

### Example 4: Development to Production Pipeline

**Development Environment**

```bash
# Developers work with human-readable files
vim configs/production.gbln

# Validate and auto-fix formatting before commit
gbln validate --fix configs/production.gbln

# Commit to Git (only .gbln files)
git add configs/production.gbln
git commit -m "Update production config"
```

**.gitignore**:
```gitignore
# GBLN I/O Files (generated, not committed)
*.io.gbln
*.io.gbln.xz
```

**CI/CD Pipeline**

```bash
# Build script (runs in CI)
#!/bin/bash

echo "Generating I/O formats..."
for file in configs/*.gbln; do
  gbln write "$file" -v
done

echo "Packaging for deployment..."
tar czf app-configs.tar.gz configs/*.io.gbln.xz

echo "File sizes:"
du -h configs/*.gbln
du -h configs/*.io.gbln.xz
```

**Production Deployment**

```bash
# Deploy only I/O formats (not source .gbln files)
scp app-configs.tar.gz prod-server:/var/lib/app/
ssh prod-server "cd /var/lib/app && tar xzf app-configs.tar.gz"

# Application reads I/O formats
systemctl restart myapp
# App loads config.io.gbln.xz (fast, efficient)
```

---

### Example 5: Runtime Config Updates

**Application Modifies Config**

```rust
use gbln::{parse_file, write_io, GblnConfig};

// Load current config
let mut value = parse_file("config.gbln")?;

// Modify at runtime
value["app"]["server"]["workers"] = Value::U8(8);  // Increase workers

// Write updated I/O format
let config = GblnConfig::default();
write_io(&value, Path::new("config.io.gbln.xz"), &config)?;

println!("Config updated at runtime");
```

**Sync Back to Source**

```bash
# Later: sync runtime changes back to source for documentation
gbln read config.gbln

# config.gbln now reflects runtime changes
git diff config.gbln
# Shows: workers changed from 4 to 8
```

---

### CLI Commands Reference

```bash
# Generate I/O format (MINI + XZ)
gbln write config.gbln
# → config.io.gbln.xz

# Generate I/O format without compression
gbln write --no-compress config.gbln
# → config.io.gbln

# Generate I/O format with pretty format (development mode)
gbln write --no-mini config.gbln
# → config.io.gbln.xz (pretty-printed, but still XZ compressed)

# Update source from I/O format
gbln read config.gbln
# Reads config.io.gbln.xz, writes config.gbln

# Verbose output (see size savings)
gbln write config.gbln -v
# Output:
#   Reading: config.gbln (287 bytes)
#   Parsing: OK (12 values)
#   MINI GBLN: 152 bytes (-47%)
#   XZ compress (level 6): 68 bytes (-76%)
#   Written: config.io.gbln.xz
```

---

## Edge Cases

### UTF-8 Characters

```gbln
:| UTF-8 Character Handling
text{
    :| ASCII (1 byte per character)
    english<s5>(Hello)                       :| 5 chars, 5 bytes
    
    :| Latin Extended (1-2 bytes)
    french<s6>(Café)                         :| 4 chars, 5 bytes
    german<s8>(Grüße)                        :| 5 chars, 7 bytes
    
    :| Cyrillic (2 bytes per character)
    russian<s12>(Привет)                     :| 6 chars, 12 bytes
    
    :| Chinese/Japanese/Korean (3 bytes typically)
    chinese<s2>(北京)                        :| 2 chars, 6 bytes
    japanese<s4>(東京)                       :| 2 chars, 6 bytes
    
    :| Emoji (4 bytes typically)
    emoji<s6>(Hello🔥)                       :| 6 chars, 10 bytes
    flags<s4>(🇬🇧🇩🇪)                         :| 2 chars (flag = 2 code points each)
    
    :| Mixed
    mixed<s16>(Café 北京 🔥)                 :| 9 chars (incl spaces)
}
```

### Escape Sequences

```gbln
:| Escape Sequence Examples
escaped{
    :| Backslash
    path<s128>(C:\\Users\\Alice\\Documents)
    
    :| Newlines
    multiline<s256>(Line 1\nLine 2\nLine 3)
    
    :| Tabs
    formatted<s64>(Name:\tAlice\tAge:\t25)
    
    :| Carriage returns
    windows_line<s32>(Line 1\r\nLine 2)
    
    :| Parentheses
    formula<s64>(Value is: \(x + 1\))
    nested<s64>(Outer \(Inner \(Deep\)\))
}
```

### Special Values

```gbln
:| Special Numeric Values
special{
    :| Infinity
    positive_infinity<f32>(inf)
    negative_infinity<f32>(-inf)
    
    :| Not a Number
    not_a_number<f32>(nan)
    
    :| Zero
    positive_zero<f32>(0.0)
    negative_zero<f32>(-0.0)
    
    :| Scientific notation
    large<f64>(1.23e308)
    small<f64>(1.23e-308)
}
```

### Nested Parentheses and Angle Brackets

```gbln
:| No Escaping Needed
code{
    :| Angle brackets in values
    html<s256>(<h1>Hello</h1><p>World</p>)
    xml<s512>(<user><name>Alice</name><age>25</age></user>)
    generic<s64>(Vec<HashMap<String, Value>>)
    comparison<s32>(x < 10 && y > 5)
    
    :| Nested parentheses
    formula<s64>(f(x) = (x + 1) * (x - 1))
    lisp<s256>((lambda (x) (* x x)) 5)
    nested<s128>(outer(inner(deepest(x))))
}
```

### Empty Values

```gbln
:| Empty Values
empty{
    :| Empty strings (valid)
    empty_string<s32>()
    
    :| Null values
    null_empty<n>()
    null_explicit<n>(null)
    null_n<n>(n)
    
    :| Empty objects
    empty_object{}
    
    :| Empty arrays
    empty_typed<i32>[]
    empty_mixed[]
    empty_objects[]
}
```

---

## Best Practises

### 1. Choose Appropriate Type Sizes

**✅ Good:**
```gbln
user{
    age<i8>(25)              :| -128 to 127 is sufficient
    id<u32>(12345)           :| Large enough for millions of users
    email<s64>(alice@x.com)  :| Reasonable email length
}
```

**❌ Bad:**
```gbln
user{
    age<i64>(25)             :| Wasteful: i8 would suffice
    id<u8>(12345)            :| Too small: will overflow
    email<s1024>(alice@x.com) :| Unnecessarily large
}
```

### 2. Use Comments for Clarity

**✅ Good:**
```gbln
:| Production Configuration - Updated 2025-01-21
server{
    port<u16>(8080)          :| HTTP port
    workers<u8>(4)           :| CPU cores
    timeout_ms<u32>(30000)   :| 30 seconds
}
```

**❌ Bad:**
```gbln
server{
    port<u16>(8080)
    workers<u8>(4)
    timeout_ms<u32>(30000)
}
```

### 3. Consistent Formatting

**✅ Good:**
```gbln
user{
    id<u32>(123)
    name<s32>(Alice)
    age<i8>(25)
}
```

**❌ Bad (inconsistent spacing):**
```gbln
user{
id<u32>(123)
    name<s32>(Alice)
  age<i8>(25)
}
```

### 4. Logical Grouping

**✅ Good:**
```gbln
server{
    :| Network settings
    host<s64>(example.com)
    port<u16>(8080)
    
    :| Performance settings
    workers<u8>(4)
    timeout_ms<u32>(30000)
}
```

### 5. Use Homogeneous Arrays When Possible

**✅ Good (simpler):**
```gbln
tags<s16>[rust python golang]
```

**⚠️ Acceptable but verbose:**
```gbln
tags[
    <s16>(rust)
    <s16>(python)
    <s16>(golang)
]
```

---

## Anti-Patterns

### ❌ Using Oversized Types

```gbln
:| Don't use i64 for small numbers
age<i64>(25)             :| Use i8 instead
status_code<i64>(200)    :| Use u16 instead
```

### ❌ Inconsistent Type Choices

```gbln
:| Don't mix type sizes arbitrarily
user{
    id<u8>(123)          :| Too small
    friend_id<u64>(456)  :| Too large for same domain
}
```

### ❌ Missing Comments for Non-Obvious Values

```gbln
:| Don't omit comments for magic numbers
timeout<u32>(30000)      :| What unit? Seconds? Milliseconds?
```

**Better:**
```gbln
timeout_ms<u32>(30000)   :| 30 seconds in milliseconds
```

### ❌ Deeply Nested Structures Without Reason

```gbln
:| Don't nest unnecessarily
config{
    settings{
        options{
            values{
                items{
                    data{
                        value<i32>(42)  :| Too deep!
                    }
                }
            }
        }
    }
}
```

**Better:**
```gbln
config{
    value<i32>(42)
}
```

---

*GBLN Examples v1.0*  
*© 2025 Vivian Voss - Apache 2.0 License*

---

**End of Examples**
