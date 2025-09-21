# env-run-pwsh: Cross-Platform Environment Management in PowerShell

## Overview

[env-run-pwsh](https://github.com/nathanfox/env-run-pwsh) is a PowerShell-based environment management utility that provides consistent environment variable management across development, UAT, and production environments. The tool is a cross-platform PowerShell port of the original bash-based env-run utility, designed to work with PowerShell 7+ on Windows, Linux, and macOS.

## Core Functionality

The utility operates in two primary modes:

1. **Environment Loading**: Loads environment variables into the current PowerShell session
2. **Command Execution**: Runs commands with environment-specific variables applied

Environment configurations are layered, with a base configuration file providing shared settings and environment-specific files (dev, uat, prod) providing overrides. This approach reduces duplication and maintains consistency across environments.

## Architecture

### Configuration Storage

Environment files are stored in platform-appropriate locations:
- Windows: `%APPDATA%\env\`
- Linux/macOS: `$XDG_CONFIG_HOME/env/` or `~/.config/env/`

The standard configuration structure includes:
- `base.env`: Shared variables across all environments
- `dev.env`: Development-specific overrides
- `uat.env`: UAT/staging-specific overrides
- `prod.env`: Production-specific overrides

### Implementation Details

The main script (`env-run.ps1`) performs the following operations:

1. **Platform Detection**: Determines the appropriate configuration directory based on the operating system
2. **File Parsing**: Reads and parses environment files, handling comments, empty lines, and quoted values
3. **Variable Merging**: Combines base and environment-specific variables with proper override precedence
4. **Environment Application**: Sets process-level environment variables
5. **Command Execution**: Optionally executes commands with the loaded environment

The implementation uses PowerShell's native capabilities for cross-platform compatibility, avoiding platform-specific shell commands where possible.

## Secret Management Integration

The utility supports multiple secret storage backends:

### PowerShell SecretManagement Module
PowerShell's official secret management module provides a unified interface for various secret vaults. Secrets can be retrieved dynamically in environment files using PowerShell command substitution.

### Platform-Specific Solutions
- **Linux**: Integration with libsecret via `secret-tool`
- **macOS**: Direct access to Keychain via `security` command
- **Windows**: Windows Credential Manager via `cmdkey`

Secret retrieval commands can be embedded in environment files using command substitution syntax, allowing sensitive values to be stored securely outside the configuration files.

## Setup and Configuration

The `setup-envs.ps1` script automates initial configuration:

1. Creates the appropriate directory structure
2. Generates template environment files with example configurations
3. Detects available secret management tools on the system
4. Provides platform-specific guidance for secret storage setup

The setup script includes comprehensive detection of available secret management tools and provides specific installation instructions when tools are missing.

## Testing Infrastructure

The project includes two comprehensive test suites:

### Test-EnvRun.ps1
Tests the main utility functionality:
- Error handling for missing files and invalid environments
- Environment variable loading and override behavior
- Command execution with proper argument passing
- Environment loading without command execution
- Export functionality for generating PowerShell variable assignments

### Test-SetupEnvs.ps1
Validates the setup script:
- Directory creation across platforms
- File generation and content validation
- Cross-platform path handling
- Secret tool detection accuracy
- Idempotent behavior on repeated runs

Both test suites use isolated test environments to avoid interference with existing configurations.

## Usage Patterns

### Basic Environment Loading
```powershell
./env-run.ps1 dev
```
This loads development environment variables into the current PowerShell session.

### Command Execution
```powershell
./env-run.ps1 prod npm start
./env-run.ps1 uat dotnet test
```
Commands execute with the specified environment's variables applied.

### Export Mode
```powershell
./env-run.ps1 dev -Export
```
Outputs PowerShell variable assignment statements for manual execution or script generation.

## Comparison with Original env-run

Key differences from the bash-based original:

- **PowerShell Native**: Uses PowerShell cmdlets and syntax throughout
- **Cross-Platform by Design**: Single implementation works across all supported platforms
- **Enhanced Secret Management**: Multiple backend support with automatic detection
- **Structured Error Handling**: PowerShell's exception handling provides detailed error information
- **Parameter Validation**: Built-in parameter sets and validation attributes

## Technical Requirements

- PowerShell 7.0 or higher (pwsh command)
- Platform-appropriate secret management tools (optional)
- Write access to configuration directory

## Platform Considerations

### Windows
- Uses Windows-style paths internally but accepts Unix-style paths
- Credential Manager integration requires appropriate permissions
- Environment variables set at process level only

### Linux
- Respects XDG Base Directory specification
- libsecret integration requires D-Bus session
- File permissions automatically set to user-only access

### macOS
- Keychain access may require user authorization
- Supports both Intel and Apple Silicon architectures
- Configuration stored in standard macOS locations

## Security Considerations

- Environment files should not contain sensitive values directly
- Secret management tools should be used for credentials and keys
- File permissions restrict access to the current user
- Process-level variables prevent persistence beyond session
- Command substitution allows dynamic secret retrieval without storage

## Performance Characteristics

- Command execution adds negligible latency
- Secret retrieval time depends on the backend used
- File parsing optimized for typical environment file sizes
- No background processes or persistent services required