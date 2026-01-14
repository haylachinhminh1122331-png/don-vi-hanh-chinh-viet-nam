# Vietnamese Administrative Units Database

## Project Overview
This repository contains a MySQL database dump of Vietnam's administrative divisions (provinces, districts, wards/commune) sourced from the General Statistics Office (GSO). Data last updated October 2017.

## Database Schema

### Hierarchical Structure
Vietnam's administrative divisions follow a 3-level hierarchy:
- **Province** (Tỉnh/Thành phố Trung ương) - Top level
- **District** (Huyện/Quận/Thị xã) - Middle level
- **Ward/Commune** (Phường/Xã/Thị trấn) - Bottom level

### Tables

#### `province`
```sql
CREATE TABLE province (
  id char(6) PRIMARY KEY,        -- 6-char code (e.g., '01' for Hà Nội)
  name varchar(45) NOT NULL,     -- Full name in Vietnamese
  type varchar(45) NOT NULL      -- 'Tỉnh' or 'Thành phố Trung ương'
);
```

#### `district`
```sql
CREATE TABLE district (
  id char(6) PRIMARY KEY,        -- 6-char code
  name varchar(45) NOT NULL,     -- Full name in Vietnamese
  type varchar(45) NOT NULL,     -- 'Huyện', 'Quận', or 'Thị xã'
  province_id char(6) NOT NULL   -- References province.id
);
```

#### `ward`
```sql
CREATE TABLE ward (
  id char(6) PRIMARY KEY,        -- 6-char code
  name varchar(45) NOT NULL,     -- Full name in Vietnamese
  type varchar(45) NOT NULL,     -- 'Phường', 'Xã', or 'Thị trấn'
  district_id char(6) NOT NULL   -- References district.id
);
```

## Key Patterns

### ID Format
- All administrative units use 6-character string IDs
- IDs are not auto-incrementing integers but meaningful codes
- Example: Hà Nội province = '01', Ba Đình district = '001'

### Naming Conventions
- All names are in Vietnamese with proper diacritics
- Use UTF-8 encoding with `utf8_unicode_ci` collation
- Names include administrative type prefixes (e.g., "Quận Ba Đình", "Huyện Sóc Sơn")

### Data Relationships
- Foreign keys establish hierarchy: ward → district → province
- No circular references or complex relationships
- Data is read-only (reference data from GSO)

## Working with the Data

### Importing the Database
```bash
mysql -u username -p database_name < dvhcvn_mysql.sql
```

### Common Queries
```sql
-- Get all provinces
SELECT * FROM province ORDER BY name;

-- Get districts in a province
SELECT d.* FROM district d
JOIN province p ON d.province_id = p.id
WHERE p.name = 'Thành phố Hà Nội';

-- Get full address hierarchy
SELECT
  p.name as province,
  d.name as district,
  w.name as ward
FROM ward w
JOIN district d ON w.district_id = d.id
JOIN province p ON d.province_id = p.id
WHERE w.id = '00001';
```

## Development Notes

### Data Updates
- Source data comes from GSO website: http://www.gso.gov.vn/dmhc2015/
- Updates are manual - check GSO for newer administrative changes
- Vietnam occasionally reorganizes administrative boundaries

### Character Encoding
- Always use UTF-8 encoding
- Preserve Vietnamese diacritics in all operations
- Database collation: `utf8_unicode_ci`

### No Build Process
This is a data-only repository. No compilation, testing, or deployment workflows exist.