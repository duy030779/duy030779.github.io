# Enhanced Filter Functions - JavaScript to Python Conversion

## Overview

This document explains the enhanced filter functionality that has been converted from the `func.js` JavaScript file to Python for use in the `schedule_bi.py` system. The new filter functions provide much more sophisticated condition processing for the `condition_type` field in WARNING schedules.

## What Was Added

### 1. Enhanced Filter Functions
The JavaScript `filterFunc` object has been converted to Python as `FILTER_FUNCTIONS`, supporting 22 different condition types:

**Text Operations:**
- `text is` - Exact text match
- `text is not` - Text not equal
- `includes` - Contains substring (case-insensitive)
- `start` - Starts with text
- `end` - Ends with text

**List Operations:**
- `in` - Value in list
- `not in` - Value not in list
- `value` - Dashboard filter (with exclude support)

**Comparison Operations:**
- `=`, `!=`, `<`, `<=`, `>`, `>=` - Standard comparisons
- `between` - Range between two values
- `after`, `before` - Date/number comparisons

**Null Checks:**
- `is null` - Check for null values
- `is not null` - Check for non-null values

**Date Operations:**
- `date is` - Date equality with type support
- `time` - Time-based filtering
- `range-value` - Value range filtering

### 2. Smart Data Type Handling
The system now automatically handles different data types:

**String Types:** `['char', 'text']`
- Automatically escapes single quotes
- Wraps values in quotes for SQL

**Date Types:** `['time', 'date', 'interval']`
- Supports multiple date storage formats
- Converts between date/timestamp/string formats

**Number Types:** `['int', 'integer', 'float', 'double', 'decimal', 'number', 'numeric']`
- No quote wrapping for numeric values
- Proper type casting

### 3. Advanced SQL Generation
The new `apply_filter_condition()` function generates proper SQL WHERE clauses with:
- Automatic SQL injection protection
- Proper quote escaping
- Type-aware value formatting
- Error handling and fallback logic

## How It Works

### Before (Simple Processing)
```python
# Old basic condition processing
if condition_type.lower() in ['is null', 'is not null']:
    where_condition = f"{column_id} {condition_type}"
elif condition_value is not None and condition_value != '':
    if type_data == 'date':
        formatted_value = f"'{condition_value}'"
    elif isinstance(condition_value, str):
        formatted_value = f"'{condition_value}'"
    else:
        formatted_value = str(condition_value)
    where_condition = f"{column_id} {condition_type} {formatted_value}"
```

### After (Enhanced Processing)
```python
# New enhanced condition processing
where_condition = apply_filter_condition(
    column_id=column_id,
    condition_type=condition_type,
    condition_value=condition_value,
    type_data=type_data,
    excluded=excluded
)
```

## Usage Examples

### 1. Text Filtering
```python
# Exact match
apply_filter_condition('name', 'text is', 'John Doe', 'string')
# Result: name = 'John Doe'

# Contains
apply_filter_condition('email', 'includes', '@company.com', 'string')
# Result: LOWER(email) LIKE LOWER('%@company.com%')

# Starts with
apply_filter_condition('title', 'start', 'Mr.', 'string')
# Result: title LIKE 'Mr.%'
```

### 2. List Operations
```python
# In list
apply_filter_condition('status', 'in', ['active', 'pending'], 'string')
# Result: status IN ('active', 'pending')

# Not in list
apply_filter_condition('category', 'not in', ['deleted'], 'string')
# Result: category NOT IN ('deleted')
```

### 3. Numeric Comparisons
```python
# Between range
apply_filter_condition('age', 'between', [18, 65], 'number')
# Result: age BETWEEN 18 AND 65

# Greater than
apply_filter_condition('score', '>', 80, 'number')
# Result: score > 80
```

### 4. Date Operations
```python
# Date equality
apply_filter_condition('created_date', 'date is', '2025-08-04', 'date')
# Result: created_date = '2025-08-04'

# Date with timestamp storage
apply_filter_condition('created_date', 'date is', '2025-08-04', 'number')
# Result: created_date = 1722729600000
```

### 5. Null Checks
```python
# Is null
apply_filter_condition('phone', 'is null', None, 'string')
# Result: phone IS NULL

# Is not null
apply_filter_condition('email', 'is not null', None, 'string')
# Result: email IS NOT NULL
```

## Integration with conditionSend

The enhanced filter system is now integrated into the WARNING schedule processing. When processing `conditionSend` conditions, the system:

1. **Extracts condition parameters:**
   - `id` (column_id)
   - `type` (condition_type) 
   - `value` (condition_value)
   - `typeData` (type_data)
   - `excluded` (excluded flag)

2. **Applies the filter function:**
   ```python
   where_condition = apply_filter_condition(
       column_id=column_id,
       condition_type=condition_type,
       condition_value=condition_value,
       type_data=type_data,
       excluded=excluded
   )
   ```

3. **Builds the final query:**
   ```sql
   SELECT * FROM (
       {original_query}
   ) AS test
   WHERE (
       {generated_conditions}
   ) LIMIT 10
   ```

## Error Handling

The system includes comprehensive error handling:

- **Unknown condition types**: Falls back to simple comparison
- **Invalid values**: Logs errors and uses safe defaults
- **SQL injection protection**: Automatic quote escaping
- **Type conversion errors**: Graceful fallback to string handling

## Benefits

1. **More Sophisticated Filtering**: Support for 22+ condition types vs basic equality
2. **Better SQL Generation**: Proper type handling and SQL injection protection
3. **JavaScript Compatibility**: Matches the frontend filter behavior exactly
4. **Extensible**: Easy to add new filter types as needed
5. **Robust Error Handling**: Graceful degradation when issues occur
6. **Better Logging**: Detailed logging for debugging and monitoring

## Testing

Run the test script to see all filter functions in action:

```bash
python test_filter_functions.py
```

This will demonstrate all supported condition types and show the generated SQL for each case.

## Migration Notes

The update is **backward compatible**. Existing schedules will continue to work, but they will now benefit from:

- Better SQL injection protection
- More accurate type handling
- Improved error handling
- Enhanced logging

New schedules can take advantage of all the advanced filter types supported by the frontend BI system.
