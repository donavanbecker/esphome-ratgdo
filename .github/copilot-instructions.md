# ESPHome ratgdo Garage Door Controller

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

ESPHome ratgdo is a firmware project for controlling garage door openers using ratgdo control boards. It supports multiple board versions (v2.0, v2.5, v2.5i, v32) and garage door protocols (Security+ 1.0, Security+ 2.0, Dry Contact).

## Working Effectively

### Bootstrap and Test the Repository
- `cd /path/to/esphome-ratgdo`
- `python3 --version` -- verify Python 3.11+ is available
- `pip3 install -r requirements-test.txt` -- installs pytest, coverage, and pyyaml dependencies (takes 30-60 seconds)
- `python -m pytest tests/ -v` -- runs test suite in 0.3 seconds. ALWAYS run this before making changes.
- `python3 scripts/update_refs_for_ci.py` -- validates CI reference update script works (takes 0.03 seconds)

### Development Workflow
- ALWAYS run `python -m pytest tests/ -v` before and after making changes (takes 0.29 seconds)
- Test changes to CI script: `python3 scripts/update_refs_for_ci.py` (takes 0.035 seconds)
- Serve web installer locally: `cd static && python3 -m http.server 8000` then visit http://localhost:8000
- Pre-commit hooks exist but may fail due to network timeouts -- install with `pip3 install pre-commit && pre-commit install`

### Build Process (CI Only)
The firmware builds ONLY run in CI using GitHub Actions. Local builds are NOT supported due to:
- Requires `ratgdo/esphome-build-action@main` custom action
- Builds 16+ firmware variants in parallel  
- Each firmware build takes approximately 5-15 minutes
- NEVER CANCEL: Total CI build time is 20-45 minutes. Set timeout to 60+ minutes when testing CI changes.

To test build changes:
- Make changes to YAML configuration files
- Create pull request -- this triggers CI builds
- NEVER CANCEL the CI build process, builds take 20-45 minutes

## Validation

### Manual Testing Requirements
- ALWAYS run the complete test suite after making changes: `python -m pytest tests/ -v`
- Test web installer functionality: start local server and verify HTML loads correctly
- For YAML configuration changes: Basic syntax checking not possible due to ESPHome-specific `!lambda` tags. Test in CI instead.
- Validate CI script changes by running `python3 scripts/update_refs_for_ci.py` and checking git diff output

### Known Working Commands
All commands tested and validated to work with measured timing:

```bash
# Dependencies and testing (takes 30-60 seconds)
pip3 install -r requirements-test.txt

# Run tests (takes 0.29 seconds, 11 tests pass)  
python -m pytest tests/ -v

# Validate CI script (takes 0.035 seconds)
python3 scripts/update_refs_for_ci.py

# Serve web installer (instant, serves 11.8KB HTML file)
cd static && python3 -m http.server 8000
```

### Commands That Do NOT Work Locally
- `esphome` builds -- only work in CI environment with custom action
- `pre-commit run --all-files` -- may timeout due to network restrictions
- Installing ESPHome locally -- network timeouts prevent installation

## Common Tasks

### Repository Structure
```
/home/runner/work/esphome-ratgdo/esphome-ratgdo/
├── .github/workflows/build.yml  # CI build pipeline
├── components/ratgdo/           # Custom ESPHome component source
├── static/                      # Web installer and configurations
│   ├── index.html              # Web installer interface
│   ├── *.yaml                  # Board-specific configurations
│   ├── wiring_diagrams/        # Hardware wiring diagrams
│   └── webui_documentation.html # Device web UI documentation
├── tests/                      # Python test suite
├── scripts/                    # CI utility scripts
├── base*.yaml                  # Base configuration templates
├── v*board*.yaml               # Board-specific configurations (symlinks to static/)
├── requirements-test.txt       # Python test dependencies
├── .pre-commit-config.yaml     # Code formatting configuration
└── library.json               # PlatformIO library metadata
```

### Common File Outputs
Running `ls -la` in repository root shows:
- Multiple `v*board*.yaml` files are symlinks pointing to `static/` directory
- Main configurations: `base.yaml`, `base_secplusv1.yaml`, `base_drycontact.yaml` 
- Development files: `.pre-commit-config.yaml`, `requirements-test.txt`
- All board configurations exist both as symlinks in root and actual files in `static/`

### Key Configuration Files
- `base.yaml` - Main configuration template for Security+ 2.0
- `base_secplusv1.yaml` - Configuration template for Security+ 1.0  
- `base_drycontact.yaml` - Configuration template for dry contact systems
- `static/v*board*.yaml` - Board-specific configurations combining base + hardware settings

### Board Variants and Protocols
The project supports these combinations:
- **v2.0 boards**: ESP8266/ESP32 D1 Mini (Security+ 2.0 only)
- **v2.5 boards**: ESP8266/ESP32 D1 Mini (Security+ 1.0 and 2.0)
- **v2.5i boards**: Integrated ESP32 (Security+ 1.0, 2.0, and Dry Contact)
- **v32 boards**: ESP32-based (Security+ 1.0, 2.0, and Dry Contact)

### File Naming Convention
- `v{version}board_{chip}_{variant}.yaml` for development boards
- `v{version}board_{chip}_{variant}_secplusv1.yaml` for Security+ 1.0
- No suffix = Security+ 2.0
- `_drycontact` suffix = Dry contact mode

### CI Pipeline Details
Located in `.github/workflows/build.yml`:
1. **Test Phase**: Runs Python tests (takes 1-2 seconds)
2. **Build Phase**: Builds all firmware variants in parallel (20-45 minutes total)
   - Uses `ratgdo/esphome-build-action@main`
   - Builds 16+ different firmware configurations
   - Each individual build takes 5-15 minutes
   - NEVER CANCEL: Always wait for completion
3. **Deploy Phase**: Updates GitHub Pages with web installer

### Python Component Structure
- `components/ratgdo/` contains the custom ESPHome component
- Written in C++ with Python integration layer
- Implements Security+ 1.0/2.0 protocols and dry contact control
- Provides entities for: cover (door), lock, lights, sensors, switches

### Testing Framework
- Uses pytest for Python tests
- Tests located in `tests/scripts/`
- Currently tests the CI reference update script functionality
- Tests run in under 2 seconds
- ALWAYS run tests before committing changes

### Web Installer
- Located in `static/index.html`  
- Allows users to flash firmware via web browser
- Includes hardware selection and protocol configuration
- Served from GitHub Pages at ratgdo.github.io/esphome-ratgdo/
- Test locally: `cd static && python3 -m http.server 8000`

### Network Limitations
This development environment has network restrictions that prevent:
- Installing ESPHome and related build tools
- Running pre-commit hooks (due to PyPI timeouts)
- Installing additional Python packages beyond basic requirements

### Making Changes
1. **For configuration changes**: Edit YAML files, test syntax, run tests
2. **For component code changes**: Edit C++ files in `components/ratgdo/`
3. **For CI changes**: Edit `scripts/update_refs_for_ci.py`, test locally, run test suite  
4. **For web installer changes**: Edit `static/index.html`, test with local server

### Troubleshooting
- If tests fail: Check Python syntax and imports
- If CI script fails: Run `python3 scripts/update_refs_for_ci.py` and check git diff
- If YAML changes break CI: ESPHome YAML uses special syntax (`!lambda`, etc.) that standard YAML parsers cannot validate
- If web installer breaks: Test with local HTTP server

CRITICAL: Never attempt to cancel or interrupt CI builds. Firmware builds require 20-45 minutes to complete and must run to completion for valid results.