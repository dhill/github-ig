# FHIR IG Automatic Publishing Setup

This repository is configured to automatically build and publish your FHIR Implementation Guide (IG) to GitHub Pages whenever changes are pushed to the main/master branch.

## How It Works

The GitHub Actions workflow (`.github/workflows/publish.yml`) automatically:

1. **Installs Dependencies**: Sets up Java, Node.js, SUSHI, and Jekyll
2. **Updates IG Publisher**: Downloads the latest HL7 FHIR IG Publisher
3. **Runs SUSHI**: Converts FSH files to FHIR resources
4. **Builds the IG**: Generates the complete Implementation Guide with all web pages
5. **Deploys to gh-pages**: Publishes the `output` directory to the `gh-pages` branch

## Initial Setup Required

To enable automatic publishing, you need to configure GitHub Pages for your repository:

### Step 1: Enable GitHub Actions (if not already enabled)

1. Go to your repository on GitHub
2. Click on **Settings** tab
3. Click on **Actions** > **General** in the left sidebar
4. Ensure "Allow all actions and reusable workflows" is selected
5. Under "Workflow permissions", select "Read and write permissions"
6. Check "Allow GitHub Actions to create and approve pull requests"
7. Click **Save**

### Step 2: Create gh-pages Branch (Optional)

The GitHub Actions workflow will automatically create the `gh-pages` branch on the first run. However, if you prefer to create it manually first:

```bash
# Create an empty orphan branch
git checkout --orphan gh-pages

# Remove all files from staging
git rm -rf .

# Create an initial commit
git commit --allow-empty -m "Initialize gh-pages branch"

# Push to GitHub
git push origin gh-pages

# Switch back to main branch
git checkout main
```

Alternatively, you can skip this step and let the workflow create it automatically.

### Step 3: Configure GitHub Pages

1. Go to your repository on GitHub
2. Click on **Settings** tab
3. Click on **Pages** in the left sidebar
4. Under "Build and deployment":
   - **Source**: Select "Deploy from a branch"
   - **Branch**: Select `gh-pages` and `/ (root)` folder
     - Note: If the branch doesn't exist yet, run the workflow first (Step 4) and then return here to configure
   - Click **Save**

### Step 4: Trigger the First Build

Push any change to the main/master branch or manually trigger the workflow:

```bash
git add .
git commit -m "Enable automatic IG publishing"
git push origin main
```

Alternatively, manually trigger the workflow:
1. Go to the **Actions** tab in your repository
2. Click on "Build and Publish FHIR IG"
3. Click "Run workflow"
4. Select the branch and click "Run workflow"

### Step 5: Access Your Published IG

After the workflow completes (typically 5-15 minutes), your IG will be available at:

```
https://[username].github.io/[repository-name]/
```

For this repository: `https://dhill.github.io/github-ig/`

## Monitoring Builds

- Go to the **Actions** tab in your repository to see workflow runs
- Click on a specific run to view detailed logs
- If a build fails, check the logs to identify the issue

## Customization

### Changing Trigger Branches

Edit `.github/workflows/publish.yml` to change which branches trigger builds:

```yaml
on:
  push:
    branches:
      - main        # Trigger on main branch
      - develop     # Add other branches as needed
```

### Building on Pull Requests

To test builds on PRs without deploying:

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

Then modify the deploy step to only run on push events:

```yaml
- name: Deploy to GitHub Pages
  if: github.event_name == 'push'
  uses: peaceiris/actions-gh-pages@v3
  # ... rest of the configuration
```

## Troubleshooting

### Build Fails with Java Errors

- The workflow uses Java 17. Most FHIR IG Publisher versions work with Java 11-17
- Check the Actions log for specific error messages

### SUSHI Errors

- Validate your FSH files locally using `sushi .`
- Check `input/fsh/` for syntax errors

### Publisher Errors

- Review warnings in the Actions log
- Some warnings are acceptable; errors will prevent deployment

### gh-pages Branch Not Created

- Ensure GitHub Actions has write permissions (Step 1 above)
- Check if the workflow completed successfully in the Actions tab
- The first run creates the `gh-pages` branch automatically using `force_orphan: true`
- If you prefer to create it manually first, see Step 2 above

### Page Not Accessible

- Wait a few minutes after the first deployment
- Verify GitHub Pages is enabled and configured correctly (Step 2)
- Check that your repository is public (or you have GitHub Pro/Team for private repos)

## Local Development

To build locally before pushing:

```bash
# Update publisher
./_updatePublisher.sh

# Run SUSHI
sushi .

# Build IG
./_genonce.sh

# View output
open output/index.html  # macOS
# or
xdg-open output/index.html  # Linux
# or
start output/index.html  # Windows
```

## Additional Resources

- [FHIR Shorthand Documentation](https://fshschool.org/)
- [HL7 IG Publisher Documentation](https://confluence.hl7.org/display/FHIR/IG+Publisher+Documentation)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
