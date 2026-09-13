# Bioconductor-oriented review of `mesa`

Overall: **promising, but not fully Bioconductor-ready yet**.

## 1) S4 / Bioconductor object design

### Good

- The package does use S4 where it matters: it defines `mesaDimRed`, `mesaPCA`, and `mesaUMAP` with formal slots, `show()` methods, and validity methods (`/home/runner/work/mesa/mesa/R/classes.R:32-105`, `132-182`, `208-255`).
- It also extends `qseaSet` with S4 generics/methods like `getMart` / `setMart` (`/home/runner/work/mesa/mesa/R/qseaExtra.R:87-104`).
- It uses core Bioconductor data types like `GRanges` and builds on `qsea`, which is a sensible Bioconductor alignment.

### Concerns

- The custom S4 classes have **minimal validity** only; they do not check cross-slot consistency (for example `samples` vs `rownames(sampleTable)`, dimensions of `points`/`dataTable`, or contents of `res`) (`/home/runner/work/mesa/mesa/R/classes.R:98-105`, `176-181`, `250-254`).
- The package leans heavily on **S3 dplyr methods on `qseaSet`** (`/home/runner/work/mesa/mesa/NAMESPACE:3-10`), which is user-friendly but less Bioconductor-idiomatic than a richer accessor/coercion layer.
- The package help page is marked internal (`/home/runner/work/mesa/mesa/R/mesa-package.R:1-3`), which is not ideal for a Bioconductor-facing package.

### Assessment

- **Acceptable foundation**, but I would strengthen class validity and accessor design before submission.

## 2) Documentation / scientific framing

### Good

- The README clearly states the biological problem and assay types (`/home/runner/work/mesa/mesa/README.md:3-13`).
- The vignettes are the strongest part: the introduction and generation vignettes explain the **scientific purpose**, assay context, workflow, and interpretation in human terms (`/home/runner/work/mesa/mesa/vignettes/introduction.Rmd:23-40`, `71-90`; `/home/runner/work/mesa/mesa/vignettes/generation.Rmd:24-84`, `106-173`).
- Function-level docs are detailed and oriented to users, especially `makeQset()` (`/home/runner/work/mesa/mesa/R/makeQset.R:1-213`).

### Concerns

- The package-level description is still fairly terse/technical and does not explain the scientific niche as well as the vignettes do (`/home/runner/work/mesa/mesa/man/mesa-package.Rd:7-10`).
- README says “a full vignette is under construction” even though there are already multiple vignettes (`/home/runner/work/mesa/mesa/README.md:13`).
- There are noticeable wording and grammar issues in vignettes, which Bioconductor reviewers often notice (`/home/runner/work/mesa/mesa/vignettes/introduction.Rmd:24`, `32`, `43`).

### Assessment

- **Strong overall**, especially on scientific explanation, but the package-level landing pages need polishing.

## 3) Test coverage

### Good

- There is broad topical coverage: DMRs, qset editing, PCA/UMAP, annotation, parallelism, utility functions, and construction workflows (`/home/runner/work/mesa/mesa/tests/testthat/`).
- Several tests check both success and error behavior, which is good Bioconductor practice (for example `/home/runner/work/mesa/mesa/tests/testthat/test-DMRs.R`, `/home/runner/work/mesa/mesa/tests/testthat/test-editQset.R`, `/home/runner/work/mesa/mesa/tests/testthat/test-pca.R`).
- CI is set up to run `R CMD check`, `BiocCheck`, and coverage tooling (`/home/runner/work/mesa/mesa/.github/workflows/check-bioc.yml:264-300`).

### Concerns

- The test harness sets `options(skip_long_checks = TRUE)` by default (`/home/runner/work/mesa/mesa/tests/testthat.R:11`), and many important tests immediately call `skip_long_checks()` (`/home/runner/work/mesa/mesa/R/utils.R:238-262`; for example `test-DMRs.R:3`, `test-makeQset.R:4`, `test-exampleQset.R:3`, `test-pca.R:94`). So **effective routine coverage is likely much lower than it appears**.
- Some tests depend on **local/internal absolute paths** and are skipped on CI (`/home/runner/work/mesa/mesa/tests/testthat/test-makeQset.R:87-194`), which is not a good Bioconductor story.
- Some tests require internet or external annotation resources (`/home/runner/work/mesa/mesa/tests/testthat/test-mouse.R:3-5`), which reduces reproducibility on builders.
- I did not find direct tests for **invalid construction / validity** of the custom S4 classes.

### Assessment

- **Breadth is good, but submission-grade reliability is not there yet**.

## 4) Highest-priority changes before Bioconductor submission

1. **Strengthen S4 validity and accessor coverage** for `mesaDimRed`, `mesaPCA`, and `mesaUMAP`.
2. **Promote package-level docs** from internal/terse to a clear user-facing scientific overview.
3. **Make tests builder-friendly**: reduce default skipping, remove dependence on internal file paths, and avoid network dependence where possible.
4. **Add direct tests for S4 constructors and validity failures**.
5. **Polish README and vignettes** for consistency and grammar.

## Bottom line

- **Documentation/scientific explanation:** good
- **Use of S4/Bioconductor idioms:** moderate, needs tightening
- **Test coverage:** broad in scope, but too much is skipped or environment-dependent

My overall review is: **close in spirit to Bioconductor expectations, but not yet at a submission-ready standard without object-system tightening and more robust, portable testing.**
