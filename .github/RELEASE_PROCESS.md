# Release Process

This document describes how to create a new release of rEFInd with automatic CI/CD.

## Automatic Build and Release

The project uses GitHub Actions to automatically build and release rEFInd when you push a version tag.

### Creating a Release

1. **Ensure all changes are committed and pushed to the `dev` or `main` branch**

2. **Create and push a version tag**:
   ```bash
   # Create a tag (follow existing format: v.x.y.z)
   git tag -a v.0.14.3 -m "Release v.0.14.3 with screen rotation support"
   
   # Push the tag to GitHub
   git push origin v.0.14.3
   ```

3. **GitHub Actions will automatically**:
   - Build rEFInd with all architectures
   - Create a release archive (`.tar.gz`)
   - Generate SHA256 checksum
   - Create a GitHub Release with:
     - Release notes
     - Binary archive
     - Checksum file
     - Build artifacts

4. **Check the release**:
   - Go to: https://github.com/YOUR_USERNAME/refind-code/releases
   - The new release should appear within a few minutes

### Tag Naming Convention

Follow the project's existing format with dot after 'v':

- `v.0.14.0` - Major version
- `v.0.14.1` - Minor update
- `v.0.14.2` - Patch release

### What Gets Built

The CI builds:
- `refind_x64.efi` - Main bootloader (x86-64)
- `refind_aa64.efi` - ARM64 bootloader (if applicable)
- `drivers_x64/*.efi` - Filesystem drivers (x64)
- `drivers_aa64/*.efi` - Filesystem drivers (ARM64)
- `gptsync_*.efi` - GPT sync tools

All files are packaged into a single archive.

## Continuous Integration

A single workflow handles both testing and releases:

- **Workflow**: `.github/workflows/build-release.yml`
- **Push to main/dev**: Build and test only (artifacts saved for 7 days)
- **Push tag v\*.\*.\***: Build, test, and create GitHub Release (artifacts saved for 90 days)

## Monitoring Builds

1. **Go to the Actions tab** in your GitHub repository
2. **Click on a workflow run** to see detailed logs
3. **Download artifacts** if needed

## Troubleshooting

### Build Fails

1. Check the workflow logs in GitHub Actions
2. Common issues:
   - Missing dependencies (should be auto-installed)
   - Syntax errors in code
   - Make configuration issues

### Release Not Created

1. Ensure the tag matches the pattern `v*.*.*`
2. Check that GITHUB_TOKEN has proper permissions
3. Review the workflow logs for errors

## Manual Testing

To test the workflow locally before pushing:

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y gnu-efi build-essential uuid-dev

# Build
make gnuefi

# Check outputs
ls -lh refind/refind_*.efi
```

## Updating Workflows

The workflow file is located at `.github/workflows/build-release.yml`.

It uses conditional logic (`if: startsWith(github.ref, 'refs/tags/')`) to:
- Run build tests on every push to main/dev
- Create releases only when tags are pushed

Edit this file to customize the build process.

## Example: Complete Release Flow

```bash
# 1. Make changes and commit
git add .
git commit -m "feat: some new feature"
git push origin dev

# 2. Wait for CI to pass (optional but recommended)
# Check: https://github.com/YOUR_USERNAME/refind-code/actions

# 3. Merge to main if needed
git checkout main
git merge dev
git push origin main

# 4. Create release tag
git tag -a v.0.14.3 -m "Release with screen rotation support"
git push origin v.0.14.3

# 5. Wait for release to be created (2-5 minutes)
# Check: https://github.com/YOUR_USERNAME/refind-code/releases

# 6. Done! Users can now download the release
```

## Notes

- Tags cannot be overwritten - if you need to fix a release, create a new tag
- Build time is typically 2-5 minutes
- Artifacts are automatically attached to releases
- Release notes are auto-generated from template

