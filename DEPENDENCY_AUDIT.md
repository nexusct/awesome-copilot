# Dependency Audit Report

**Date:** 2026-01-20
**Project:** awesome-copilot
**Node.js Dependencies Analyzed:** 72 total (9 production, 64 development)
**Total node_modules size:** ~25MB

---

## Summary

| Category | Status | Findings |
|----------|--------|----------|
| Security Vulnerabilities | ⚠️ Low | 4 low-severity vulnerabilities |
| Outdated Packages | ✅ Good | All direct dependencies are up-to-date |
| Dependency Bloat | ⚠️ Moderate | `all-contributors-cli` accounts for 89% of packages |

---

## 1. Security Vulnerabilities

### Vulnerability: CVE-2025-54798 (GHSA-52f5-9888-hmc6)

| Severity | Package | Description |
|----------|---------|-------------|
| Low (2.5/10) | `tmp@0.2.3` | Arbitrary temporary file/directory write via symbolic link |

**Affected Dependency Chain:**
```
all-contributors-cli@6.26.1
  └── inquirer@7.3.3
        └── external-editor@3.1.0
              └── tmp@0.0.33 (vulnerable)
```

**Risk Assessment:**
- **Impact:** Low - The vulnerability requires local access and specific conditions (symlink attack on temp directories)
- **Exploitability:** Low - Only affects temp file operations during CLI usage
- **Context:** This is a development-only dependency used for contributor management

**Resolution Options:**

1. **Wait for upstream fix** (Recommended for now)
   - The `all-contributors-cli` maintainers need to update `inquirer` from v7 to v12+
   - Current `all-contributors-cli@6.26.1` is already the latest version
   - Monitor: https://github.com/all-contributors/all-contributors-cli/issues

2. **Use npm overrides** (Temporary workaround)
   Add to `package.json`:
   ```json
   "overrides": {
     "tmp": "^0.2.4"
   }
   ```
   ⚠️ This may cause compatibility issues with `external-editor`

3. **Replace with npx usage** (See Bloat section below)

---

## 2. Outdated Packages Analysis

| Package | Current | Wanted | Latest | Status |
|---------|---------|--------|--------|--------|
| js-yaml | 4.1.1 | 4.1.1 | 4.1.1 | ✅ Up-to-date |
| vfile | 6.0.3 | 6.0.3 | 6.0.3 | ✅ Up-to-date |
| vfile-matter | 5.0.1 | 5.0.1 | 5.0.1 | ✅ Up-to-date |
| all-contributors-cli | 6.26.1 | 6.26.1 | 6.26.1 | ✅ Up-to-date |

**Conclusion:** All direct dependencies are at their latest versions.

---

## 3. Dependency Bloat Analysis

### Current State

| Category | Package Count | Size Impact |
|----------|--------------|-------------|
| Production deps | 9 packages | ~2MB |
| Dev deps (`all-contributors-cli`) | 64 packages | ~23MB |
| **Total** | **72 packages** | **~25MB** |

### Heavy Sub-dependencies from `all-contributors-cli`

| Package | Purpose | Size/Impact |
|---------|---------|-------------|
| `@babel/runtime` | Babel helpers | Heavy |
| `lodash` | Utility functions | 1.4MB (entire library) |
| `prettier` | Code formatting | Heavy |
| `inquirer` | Interactive prompts | + many deps |
| `node-fetch` | HTTP client | Moderate |
| `rxjs` | Reactive extensions | Moderate |

### Recommendation: Convert `all-contributors-cli` to npx-only usage

The `all-contributors-cli` is used infrequently (only for contributor management) and can be invoked via `npx` instead of being a permanent dev dependency.

**Current usage analysis:**
- `package.json` scripts use it directly
- `eng/contributor-report.mjs` already uses `npx all-contributors check`
- `eng/add-missing-contributors.mjs` already uses `npx all-contributors add`

**Proposed changes:**

1. Remove from `devDependencies`:
   ```diff
   - "devDependencies": {
   -   "all-contributors-cli": "^6.26.1"
   - }
   + "devDependencies": {}
   ```

2. Update npm scripts to use `npx`:
   ```json
   "scripts": {
     "contributors:add": "npx all-contributors-cli add",
     "contributors:generate": "npx all-contributors-cli generate",
     "contributors:check": "npx all-contributors-cli check"
   }
   ```

**Benefits:**
- Reduces node_modules from 72 packages to 9 packages (~87% reduction)
- Reduces disk usage from ~25MB to ~2MB
- Eliminates security vulnerabilities (they're only present when the package is installed)
- Contributors can still run the scripts (npx will download on-demand)

**Trade-offs:**
- First run of contributor commands will be slower (npx downloads on demand)
- Requires internet connection for first use

---

## 4. YAML Parser Redundancy (Minor)

The project uses two YAML parsers:

| Package | Version | Used For | Dependency Chain |
|---------|---------|----------|------------------|
| `js-yaml` | 4.1.1 | Parsing `.collection.yml` files | Direct |
| `yaml` | 2.8.1 | Frontmatter parsing | Via `vfile-matter` |

**Assessment:** This redundancy is acceptable because:
- Both are lightweight packages
- They serve different purposes (direct YAML vs frontmatter)
- Consolidating would require code changes with minimal benefit

**Optional optimization:** Could replace `js-yaml` usage with `yaml` (from vfile-matter):
```javascript
// Instead of: import yaml from "js-yaml";
import { parse } from "yaml";
```
This would eliminate one direct dependency but requires testing.

---

## 5. Recommended Actions

### High Priority

| Action | Impact | Effort |
|--------|--------|--------|
| Move `all-contributors-cli` to npx usage | Eliminates 64 packages + vulnerabilities | Low |

### Medium Priority

| Action | Impact | Effort |
|--------|--------|--------|
| Monitor `all-contributors-cli` for security updates | Addresses vulnerabilities when fixed | None |
| Add npm overrides for `tmp` if staying with installed version | Patches vulnerability | Low |

### Low Priority (Optional)

| Action | Impact | Effort |
|--------|--------|--------|
| Consolidate YAML parsers to single package | Removes 1 package | Medium |

---

## Appendix: Full Vulnerability Details

### CVE-2025-54798 Details

- **Title:** tmp allows arbitrary temporary file / directory write via symbolic link `dir` parameter
- **Severity:** Low (CVSS 2.5)
- **CWE:** CWE-59 (Improper Link Resolution Before File Access)
- **Affected Versions:** tmp <= 0.2.3
- **Fixed Version:** tmp 0.2.4
- **Advisory:** https://github.com/advisories/GHSA-52f5-9888-hmc6

---

*Report generated by dependency audit analysis*
