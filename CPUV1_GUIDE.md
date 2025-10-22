# cpuv1 Support Configuration

This directory contains a GoReleaser configuration with **cpuv1 support** for legacy x86-64 CPUs.

## 📁 Files

- **`.goreleaser.yml`** - Default configuration (modern CPUs only)
- **`.goreleaser.cpuv1.yml`** - Configuration with cpuv1 support (this file)

## 🔄 Switching Between Configurations

### Option 1: Rename Files (Simple)

To use cpuv1 support:
```bash
# Backup the current config
mv .goreleaser.yml .goreleaser.default.yml

# Use the cpuv1 config
cp .goreleaser.cpuv1.yml .goreleaser.yml

# Commit and push
git add .goreleaser.yml
git commit -m "Enable cpuv1 support for legacy CPUs"
git push
```

To switch back:
```bash
# Restore the default config
mv .goreleaser.default.yml .goreleaser.yml

git add .goreleaser.yml
git commit -m "Disable cpuv1 support"
git push
```

### Option 2: Use Different Config File (Advanced)

GitHub Actions can use a different config file:

```yaml
# In .github/workflows/release.yml
- name: Run GoReleaser
  uses: goreleaser/goreleaser-action@v6
  with:
    distribution: goreleaser
    version: '~> v2'
    args: release --clean --skip=validate --config=.goreleaser.cpuv1.yml  # ← Specify config
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
    DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Option 3: Use Branches (Recommended)

Keep configurations in separate branches:

```bash
# Main branch uses default config
git checkout master
# Uses .goreleaser.yml (no cpuv1)

# Create cpuv1 branch
git checkout -b release-with-cpuv1
cp .goreleaser.cpuv1.yml .goreleaser.yml
git add .goreleaser.yml
git commit -m "Add cpuv1 support configuration"
git push -u origin release-with-cpuv1
```

## 📊 Differences Between Configurations

### Default Configuration (`.goreleaser.yml`)

**Builds:**
- All platforms with optimal settings
- amd64 uses `GOAMD64=v3` (implicit)

**Archives (16 files):**
```
minio_Linux_x86_64.tar.gz
minio_Linux_aarch64.tar.gz
minio_Darwin_x86_64.tar.gz
... (13 more)
```

**Docker Tags:**
```
beck8/minio:latest
beck8/minio:RELEASE.xxx
beck8/minio:RELEASE.xxx-amd64
beck8/minio:RELEASE.xxx-arm64
beck8/minio:RELEASE.xxx-ppc64le
```

**Build Time:** ~30-40 minutes

---

### cpuv1 Configuration (`.goreleaser.cpuv1.yml`)

**Builds:**
- All platforms with optimal settings (`GOAMD64=v3`)
- Additional amd64 builds with `GOAMD64=v1`

**Archives (19 files):**
```
minio_Linux_x86_64.tar.gz           # v3 optimized
minio_Linux_x86_64_cpuv1.tar.gz     # v1 legacy ← NEW
minio_Linux_aarch64.tar.gz
minio_Darwin_x86_64.tar.gz          # v3 optimized
minio_Darwin_x86_64_cpuv1.tar.gz    # v1 legacy ← NEW
minio_Windows_x86_64.zip            # v3 optimized
minio_Windows_x86_64_cpuv1.zip      # v1 legacy ← NEW
... (13 more)
```

**Docker Tags:**
```
beck8/minio:latest
beck8/minio:RELEASE.xxx
beck8/minio:RELEASE.xxx-amd64
beck8/minio:RELEASE.xxx-arm64
beck8/minio:RELEASE.xxx-ppc64le
beck8/minio:RELEASE.xxx-cpuv1       ← NEW (legacy CPUs)
```

**Build Time:** ~40-50 minutes

---

## 🎯 When to Use cpuv1

### Use Default Configuration If:
- ✅ Your users have modern hardware (2013+)
- ✅ You want faster build times
- ✅ You want simpler configuration
- ✅ You're targeting cloud infrastructure

### Use cpuv1 Configuration If:
- ✅ Your users may have legacy hardware
- ✅ You need maximum compatibility
- ✅ Some users run on older virtualization platforms
- ✅ You want to fully replicate MinIO's official releases

---

## 🔍 CPU Compatibility Matrix

| CPU Micro-arch | Release Year | GOAMD64 | Supported By |
|----------------|--------------|---------|--------------|
| **Nehalem** | 2008 | v1 | cpuv1 only |
| **Sandy Bridge** | 2011 | v2 | cpuv1 only |
| **Haswell** | 2013 | v3 | Both |
| **Skylake** | 2015 | v3 | Both (default) |
| **Zen 1/2/3/4** | 2017+ | v3 | Both (default) |

### Key Instructions

| Level | Key Instructions | Performance Benefit |
|-------|------------------|---------------------|
| **v1** | SSE2 | Baseline |
| **v2** | SSE3, SSSE3, SSE4 | +10-15% |
| **v3** | AVX, AVX2, FMA | +20-30% (especially crypto/compression) |

---

## 🧪 Testing cpuv1 Locally

```bash
# Test the cpuv1 configuration
goreleaser release --snapshot --clean --skip=validate --config=.goreleaser.cpuv1.yml

# Check the generated files
ls -lh dist/

# You should see both versions:
# minio_Linux_x86_64.tar.gz        (v3)
# minio_Linux_x86_64_cpuv1.tar.gz  (v1)
```

---

## 📝 Example User Instructions

### For Users With Modern CPUs

```bash
# Download the default version (faster)
wget https://github.com/beck-8/minio/releases/download/RELEASE.xxx/minio_Linux_x86_64.tar.gz
tar -xzf minio_Linux_x86_64.tar.gz
./minio --version

# Or use Docker
docker pull beck8/minio:RELEASE.xxx
```

### For Users With Legacy CPUs

```bash
# Download the cpuv1 version (compatible)
wget https://github.com/beck-8/minio/releases/download/RELEASE.xxx/minio_Linux_x86_64_cpuv1.tar.gz
tar -xzf minio_Linux_x86_64_cpuv1.tar.gz
./minio --version

# Or use Docker
docker pull beck8/minio:RELEASE.xxx-cpuv1
```

### How to Check if You Need cpuv1

```bash
# Check CPU flags on Linux
grep -o 'avx2' /proc/cpuinfo | head -1

# If no output → you need cpuv1
# If "avx2" appears → you can use default version

# On macOS
sysctl -a | grep machdep.cpu.features | grep AVX2

# If AVX2 is listed → use default
# If not listed → use cpuv1
```

---

## 🚀 Deployment Recommendation

**Strategy A: Keep Both Configurations**
1. Keep `.goreleaser.yml` (default) in master branch
2. Keep `.goreleaser.cpuv1.yml` as a backup
3. Manually switch when needed

**Strategy B: Separate Branches (Recommended)**
1. `master` branch → `.goreleaser.yml` (default, no cpuv1)
2. `release-cpuv1` branch → `.goreleaser.yml` (with cpuv1)
3. Tag releases from the appropriate branch

**Strategy C: Always Use cpuv1**
1. Replace `.goreleaser.yml` with `.goreleaser.cpuv1.yml`
2. Delete `.goreleaser.cpuv1.yml`
3. All releases include cpuv1 support

---

## 📞 Questions?

- Default version performance: ~100% (AVX2 optimized)
- cpuv1 version performance: ~80-85% (no AVX2)
- Build time increase: ~25-30%
- Additional artifacts: +3 (Linux, macOS, Windows amd64 cpuv1)
- Additional Docker tags: +2 (Docker Hub + GHCR)

---

**Current Status:** cpuv1 support is available but not active by default.

To activate, follow the instructions in "Switching Between Configurations" above.
