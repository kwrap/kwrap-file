# kwrap-file

Kwrap's password library format

## Extension

```
.kwrap
```

## Format

```
┌─────────────┬────────────────┬──────────────────────────────────┐
│ Name        │ Length         │ Description                      │
├─────────────┼────────────────┼──────────────────────────────────┤
│ ID          │ 6 bytes        │ Fixed bytes, always '\xffKWRAP'  │
│ Version     │ 1 byte 'u8'    │ File version, now only '1'       │
│ Salt        │ 32 bytes       │ Pbkdf2 salt                      │
│ Iterations  │ 4 bytes 'u32'  │ Pbkdf2 Iterations                │
│ Data        │ ...end         │ Encrypted passwords              │
└─────────────┴────────────────┴──────────────────────────────────┘
```

## Decode

1. Read file info

```rust
let id: [u8; 6] = file.read_id();

let version: u8 = file.read_version();

let salt: [u8; 32] = file.read_salt(); 

let iterations: u32 = file.read_iterations(); 

let encrypted: Vec<u8> = file.read_to_end();
```

2. use `AES-256-GCM` to decrypt data

```rust
let key: [u8; 32] = pbkdf2("password", salt, iterations);

let data = aes_decrypt(key, encrypted);
```

3. Use `JSON` parse passwords

```rust
let passwords: Vec<PasswordData> = parse_json(data);
```

```json5
[
    // All fields are optional
    {
        "pin": 1651631704,
        "icon": "Kwrap",
        "name": "Hello World",
        "user": "kwrap",
        "email": "email@kwrap.app",
        "phone": "123456789",
        "password": "abcdefg",
        "otp": "otpauth://totp/Kwrap:user?secret=...&issuer=Kwrap",
        "links": [
            "https://kwrap.app"
        ],
        "notes": "Notes",
        "custom": [
            {
                "name": "Custom Name",
                "value": "Custom Value",
                "hidden": true
            }
        ],
        "tags": [
            "App"
        ],
        "updated": 1651631940,
        "archive": true
    },
    // ... 
]
```