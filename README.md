# Durable Semantic Data Definitions

We avoid serialization and versioning hell accross applications and over time by completely decoupling data semantics from applications and file formats.

The main idea is to define a standalone set of semantic data definitions on top of primitive data types. Each definition is uniquely identified by a never-changing (durable) globally unique id.

## Primitives

*Primitives* are definitions that DO NOT depend upon other definitions.

For example, the definition of the signed 2-complement 32-bit little-endian integer type looks as follows

```
"5ce108a4-a578-4edb-841d-068393ed93bf": {
      "name": "Int32",
      "description": "Signed 32-bit integer. 2-complement. Little endian."
    }
```

, where `5ce108a4-a578-4edb-841d-068393ed93bf` is the definition's globally unique id,
`name` is its friendly name, and `description` specifies associated semantics.

A globally unique id is defined as

```
"a81a39b0-8f61-4efc-b0ce-27e2c5d3199d": {
      "name": "Guid",
      "description": "Globally unique identifier (GUID, 16 bytes). https://tools.ietf.org/html/rfc4122."
    }
```

#### Serialization
Primitive values are serialized AS IS, without prepending the unique id.

### Arrays

Arrays are defined by simply appending `[]` to the friendly name, e.g.

```
"1cfa6f68-5b56-44a7-b4b5-bd675bc910ab": {
      "name": "Int32[]",
      "description": "Array of signed 32-bit integers. 2-complement. Little endian."
    }
```

, which states that type `Int32[]` (`1cfa6f68-5b56-44a7-b4b5-bd675bc910ab`) represents an array of values of type `Int32` (`5ce108a4-a578-4edb-841d-068393ed93bf`, see above).

#### Serialization
A binary serialization of an array starts with the length of the array given as an Int32 value (`5ce108a4-a578-4edb-841d-068393ed93bf`), followed by as many elements.

### DurableMap

A `DurableMap` is specified as

```
"f03716ef-6c9e-4201-bf19-e0cabc6c6a9a": {
      "name": "DurableMap",
      "description": "A map of key/value pairs, where keys are durable IDs with values of corresponding types."
    }
```

#### Serialization
A binary serialization of a `DurableMap` starts with the number of entries given as an Int32 value (`5ce108a4-a578-4edb-841d-068393ed93bf`), followed by as many key/value pairs, where each key is serialized as a 16-bytes unique id (`a81a39b0-8f61-4efc-b0ce-27e2c5d3199d`) and each value is serialized according to its definition.

### Structures

If a definition is composed of other definitions, this can be defined with a `struct` entry.

```
"ad8adcb6-8cf1-474e-99da-851343858935": {
      "name": "V3f",
      "description": "A 3-dimensional vector of 32-bit floats.",
      "layout": {
        "X": "Float32",
        "Y": "Float32",
        "Z": "Float32"
      }
    }
```

#### Serialization
The order of entries in the layout definition matters. If used for binary (de)serialization, then it **exactly** specifies the data layout. There is no implied or implicit padding. This means, that a binary serialization of above vector consists of exactly 12 bytes, with the first 4 bytes containing a little-endian 32-bit floating point value (`23fb286f-663b-4c71-9923-7e51c500f4ed`), followed by 4 bytes for the y-coordinate and 4 bytes for the z-coordinate.

### Semantic Definitions

The following definition gives semantic meaning to an array of 3-dim vectors of 32-bit floats:
```
"712d0a0c-a8d0-42d1-bfc7-77eac2e4a755": {
      "name": "Octree.Normals3f",
      "description": "Octree. Per-point normals (V3f[]).",
      "type": "V3f[]"
    }
```
, where `type` specifies the underlying (primitive) definition.

#### Serialization
Semantic values (values with a `type` field) are serialized by first writing the 16 bytes unique id, followed by the actual value specified in field `type` (which may itself be a semantic value, recursively).

## Rules

- a definition NEVER changes
- should there be the need to *change* a definition (a.k.a. versioning), then a new definition with a different unique id is created

## Content
Definitions are contained in `definitions.json`, which can be used for code generation and (de)serialization.