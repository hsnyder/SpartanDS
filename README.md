# SpartanDS

SpartanDS is a simple convention for storing machine learning datasets. The goal is not to provide a software library for I/O, but rather to make it possible, with nothing but a link to this page and a short text file, to unambiguously explain how a given dataset is to be accessed. The format is simple enough that I/O code can be implemented by users *ad hoc*, in order to reduce dependence on large libraries that may have environment compatibility or archival/versioning problems. Hopefully, this will help with reproducibility and collaboration. 

## Examples

### With metadata index

**Possible Folder Structure:**

```
dataset/
├── data/
│   ├── shard01/
│   │   ├── ex001.png
│   │   └── ex001.mask.npy
│   └── shard02/
│       ├── ex002.png
│       └── ex002.mask.npy
├── index.csv
└── dataset.spec
```

**dataset.spec:**

```
SpartanDSVersion = 1
Root = data
Index = index.csv
PrefixColumn = example_id
DataExtensions = .png .mask.npy

--

The `.mask.npy` file contains segmentation masks 
corresponding to each `.png` image.
```

*Note*: The prefix column (`example_id`) in `index.csv` would contain entries like:

```
shard01/ex001
shard02/ex002
```

### Without metadata index

**Possible Folder Structure:**

```
dataset/
├── ex001.png
├── ex001.json
├── ex002.png
├── ex002.json
└── dataset.spec
```

**dataset.spec:**

```
SpartanDSVersion = 1
Root = .
DataExtensions = .png .json
```


## Specification

A SpartanDS dataset consists of:

- **Data**: the actual examples for training, validation, or testing.
- An optional central **index**: a tabular file (such as CSV or Parquet) containing metadata.
- A **dataset specification file**: describing where to locate the data and index.

### Data

Each data point (training/validation/test example) consists of a group of files that share a common **prefix** (the filename portion before the first dot) but differ in their **extensions** (the filename portion from the first dot onward). For instance, `0001.json` and `0001.png` constitute a single data example with the prefix `0001`.

### Index

If a central index is provided, it must contain a column listing example prefixes (e.g., `0001`, `0002`). This column is called the **prefix column**. Prefixes can include slashes (`/`) to organize the dataset into subdirectories (e.g., `shard01/ex001`, `shard02/ex001` are distinct prefixes).

If no central index is used, the prefixes are inferred by enumerating files in the dataset's **root directory** only, ignoring any subdirectories. Thus, to use shard-style subdirectories, a central index must be provided.

The central index can be in the following standard formats (or additional custom tabular formats as needed):

- CSV or TSV (optionally gzip-compressed) with a header row. Standard extensions: `.csv`, `.tsv`, `.csv.gz`, `.tsv.gz`.
- Parquet. Standard extension: `.parquet`.
- SQLite3. Standard extensions: `.sqlite`, `.sqlite3`, `.db`.

### Dataset specification file

This plain text file consists of key-value pairs, each on a single line. Lines starting with `#` are treated as comments, as are all lines following a line beginning with `--`. Blank lines are ignored. Each key-value pair is separated by the first equal sign (`=`) on the line; whitespace surrounding keys and values is insignificant.

Standard keys and their meanings:

- **SpartanDSVersion** (mandatory): The version of this specification in use. Currently must be `1`.
- **Root** (mandatory): Path to the dataset's **root directory**, relative to the spec file's location. A value of `.` indicates the same directory as the spec file.
- **Index** (optional): Path to the metadata index file, relative to the spec file.
- **PrefixColumn** (required if an Index is specified, prohibited otherwise): The name of the column containing file prefixes in the metadata index.
- **DataExtensions** (mandatory): Space-separated list of file extensions, with one file per listed extension required for each example, all sharing the same prefix. Alternative extensions are indicated with a pipe (`|`), e.g., `DataExtensions = .jpg|.png .json .npy` means each example consists of one `.jpg` or `.png` file, one `.json` file, and one `.npy` file.

Custom keys for specific parser implementations must start with `X-`.

## Remarks

### Data archiving

If you adopt this format and plan to keep datasets around for a long period of time, it may be beneficial to take a copy of this document and store it alongside the dataset spec file. 

### Cloud storage systems

While this specification emphasizes loose files on a file system as the storage mechanism, it can still be used with object storage or even plain HTTP retrieval. While this would technically constitute an extension to this specification, there is nothing stopping particular users from using an `https://` URI in the root field, as long as their spec file parser knew how to handle it. All the other rules would remain as is. While object storage systems typically have a way of enumerating all objects at a certain prefix, plain HTTP servers often do not, or if they do, do not necessarily return the result in a parser-friendly format. Usage with a metadata index in this case is recommended so there is an explicit source  of truth for all data prefixes. 
