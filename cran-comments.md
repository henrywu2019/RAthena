## Release Summary
This release brings increase reliability when working with `AWS Athena` with a few bug fixes.

**New Features**
* When working with `AWS Athena`, `AWS` APIs can become overwhelmed and return unnecassary exceptions. To over come this `RAthena` has now been given a retry capability with exponential backoff.
* Previously `dbFetch` was restricted to only return the entire data.frame or a chunk limited to 1000 from `AWS Athena`. This was due to the restriction in the call to `AWS Athena`. Now `RAthena` uses tokens from `AWS` to iterate over. This allows `dbFetch` to back larger chunks and work similar to other DBI backend packages:

```
library(DBI)
con <- dbConnect(RAthena::athena())
res <- dbExecute(con, "select * from some_big_table limit 10000")
dbFetch(res, 5000)
```

* When appending to existing tables `dbWriteTable` now opts to use `ALTER TABLE` instead of `MSCK REPAIR TABLE` this gives an performance increase when appending onto highly partitioned tables.
* `dbWriteTable` is not compatible with `SerDes` and Data Formats

**Bug Fixes**
* 

## Examples Note:
* All R examples with `\dontrun` & `\donttest` have been given a note warning users that `AWS credentials` are required to run
* All R examples with `\dontrun` have a dummy `AWS S3 Bucket uri` example and won't run until user replace the `AWS S3 bucket uri`.

## Test environments
* local OS X install, R 3.6.1
* rhub: windows-x86_64-devel, ubuntu-gcc-release, fedora-clang-devel

## R CMD check results (local)
0 errors ✓ | 0 warnings ✓ | 0 notes ✓

## R devtools::check_rhub() results
0 errors ✓ | 0 warnings ✓ | 1 note x

Maintainer: 'Dyfan Jones <dyfan.r.jones@gmail.com>'

Number of updates in past 6 months: 8

0 errors ✓ | 0 warnings ✓ | 1 note x

**Author's Notes**
* Apologies for the fast re-submission of this package. This release contains several cost benefits for using AWS Athena. Plus a couple of bug fixes. Unit tests now increase coverage +80%.

## unit tests (using testthat) results
* OK:       112
* Failed:   0
* Warnings: 0
* Skipped:  0