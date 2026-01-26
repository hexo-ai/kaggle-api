# Kaggle CLI Examples

## Setup

Make sure you have the project downloaded and in the folder
```bash
cd /kaggle-api
source .venv/bin/activate
```

## List Kernel Versions

...?scriptVersionId=12345678 is the url associated with the version.

```bash
kaggle kernels versions alexisbcook/titanic-tutorial
```
```
Version    scriptVersionId    Date         Status
-------------------------------------------------------
1          12345678           2024-01-15   COMPLETE
2          12345679           2024-01-16   COMPLETE
```

## List Versions (CSV)

```bash
kaggle kernels versions alexisbcook/titanic-tutorial -v
```


## Pull Specific Version (by number)

```bash
kaggle kernels pull alexisbcook/titanic-tutorial --version 1 -p .
```

## Pull Specific Version (slug syntax)

```bash
kaggle kernels pull alexisbcook/titanic-tutorial/1 -p .
```                                                                                                                         
                                                                                                                                                                            
  # Download to specific folder                                                                                                                                                      
```bash
kaggle kernels pull alexisbcook/titanic-tutorial/1 -p path/to/folder
```                                                                                                                   