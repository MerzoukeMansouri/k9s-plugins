# K9s Custom Debug Plugin

A custom k9s plugin with enhanced debugging capabilities for Kubernetes containers, featuring flexible container image selection.

<video src="docs/debug-plugins.mov" width="100%" controls>
  Your browser does not support the video tag. <a href="docs/debug-plugins.mov">Download the demo video</a>.
</video>

## Table of Contents

- [About](#about)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Plugin Installation](#plugin-installation)
  - [macOS](#macos)
  - [Linux](#linux)
  - [Windows](#windows)
- [Usage](#usage)
- [Plugin Features](#plugin-features)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## About

This repository contains a custom debug plugin for [K9s](https://k9scli.io/) that allows you to launch ephemeral debug containers with your choice of container image. Perfect for troubleshooting pods without modifying the original containers.

## Features

- **Interactive Image Selection**: Choose any container image at runtime
- **Default Debian Image**: Quick debugging with Debian as the default
- **Flexible Options**: Use Alpine, Ubuntu, BusyBox, netshoot, or any custom image
- **Smart Defaults**: Press Enter to use Debian, or type your preferred image name
- **Safe Operations**: Confirmation prompt to prevent accidental execution

## Prerequisites

Before installing this plugin, ensure you have:

- **K9s installed** - [Installation guide](https://k9scli.io/)
- **kubectl 1.18+** - Required for `kubectl debug` command
- **Kubernetes cluster access** - Local or remote with valid kubeconfig
- **RBAC permissions** - Ability to create ephemeral containers

## Plugin Installation

### macOS

K9s config directory: `~/.config/k9s/` or `~/Library/Application Support/k9s/`

**Installation:**

```bash

mkdir -p ~/Library/Application\ Support/k9s/
cp plugins.yaml ~/Library/Application\ Support/k9s/
```

### Linux

K9s config directory: `~/.config/k9s/`

**Installation:**

```bash
# Create the directory if it doesn't exist
mkdir -p ~/.config/k9s/

# Copy the plugins file
cp plugins.yaml ~/.config/k9s/
```

### Windows

K9s config directory: `%LOCALAPPDATA%\k9s\` or `%APPDATA%\k9s\`

**Installation (PowerShell):**

```powershell
# Create the directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:LOCALAPPDATA\k9s"

# Copy the plugins file
Copy-Item plugins.yaml "$env:LOCALAPPDATA\k9s\"
```

**Installation (WSL):**

Follow the Linux installation instructions within your WSL distribution.

### Verify Installation

After copying the plugin file:

1. **Restart k9s** if it's already running
2. **Launch k9s**:
   ```bash
   k9s
   ```
3. **Check plugins** - Press `:` then type `plugins` to see available plugins
4. **Test the plugin** - Navigate to a pod's containers and press `Shift+D`

## Usage

### Using the Debug Plugin

1. Navigate to a pod in k9s (press `:pods` or `0`)
2. Select a container within the pod (press `c`)
3. Press `Shift+D` to activate the debug plugin
4. When prompted, enter an image name or press Enter for Debian:

```
Enter image name (default: debian):
```

#### Popular Debug Images:

- **debian** (default) - Full-featured Debian with common tools
- **alpine** - Lightweight Alpine Linux (~5MB)
- **ubuntu** - Ubuntu with apt package manager
- **busybox** - Minimal Unix utilities (~1MB)
- **nicolaka/netshoot** - Network troubleshooting tools (tcpdump, curl, dig, etc.)
- **gcr.io/google-containers/toolbox** - Google's comprehensive debug toolbox
- **curlimages/curl** - Minimal image with cURL
- **registry.k8s.io/e2e-test-images/agnhost:2.43** - Kubernetes testing tools

### Quick K9s Navigation

Essential shortcuts for using the debug plugin:

- `:pods` or `0` - Navigate to pods view
- `c` - View containers in selected pod
- `Shift+D` - **Activate debug plugin**
- `/` - Filter/search resources
- `Esc` - Go back
- `?` - Help menu

## Plugin Features

### Debug Plugin

**Shortcut**: `Shift+D`

**Description**: Launches an ephemeral debug container within a running pod, allowing you to troubleshoot issues without modifying the original container.

**Features**:

- Interactive image selection
- Default Debian image for quick access
- Shares process namespace with target container
- Full bash shell access
- Confirmation prompt to prevent accidental execution

**How it Works**:

The plugin uses `kubectl debug` under the hood:

```bash
kubectl debug -it \
  --context $CONTEXT \
  -n=$NAMESPACE $POD \
  --target=$NAME \
  --image=<selected-image> \
  --share-processes \
  -- bash
```

## Troubleshooting

### Plugin Not Working

1. **Verify plugin file location**:

   ```bash
   # Linux/macOS
   ls -la ~/.config/k9s/plugins.yaml

   # Windows
   dir %LOCALAPPDATA%\k9s\plugins.yaml
   ```

2. **Check YAML syntax**:

   ```bash
   # Install yq if not present
   brew install yq  # macOS

   # Validate YAML
   yq eval plugins.yaml
   ```

3. **Restart k9s** after configuration changes

### kubectl debug Not Available

If you get an error about `kubectl debug` not being available:

- Ensure you're using kubectl version 1.18+
- Update kubectl: `brew upgrade kubectl` (macOS) or download from [kubernetes.io](https://kubernetes.io/docs/tasks/tools/)

### Permission Issues

If you encounter permission errors:

```bash
# Check your kubeconfig
kubectl config view

# Verify cluster access
kubectl cluster-info

# Check RBAC permissions
kubectl auth can-i create pods/ephemeralcontainers
```

### Image Pull Errors

If your custom image fails to pull:

1. Verify the image name/tag is correct
2. Check if your cluster has access to the registry
3. For private registries, ensure image pull secrets are configured

### Debug Container Shell Issues

If bash is not available in your chosen image:

```yaml
# Edit plugins.yaml to use sh instead
args:
  - -c
  - |
    read -p "Enter image name (default: debian): " IMAGE
    IMAGE=${IMAGE:-debian}
    kubectl debug -it --context $CONTEXT -n=$NAMESPACE $POD --target=$NAME --image=$IMAGE --share-processes -- sh
```

### Plugin Configuration Issues

If changes don't take effect:

1. **Restart k9s** completely
2. **Check file location** - Ensure `plugins.yaml` is in the correct directory
3. **Verify syntax** - Use `yamllint plugins.yaml` or online YAML validators
4. **Check logs**:

   ```bash
   # Linux/macOS
   tail -f ~/.config/k9s/k9s.log

   # Windows
   Get-Content "$env:LOCALAPPDATA\k9s\k9s.log" -Wait
   ```

## Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests.

### Adding New Plugins

To add a new plugin, edit `plugins.yaml`:

```yaml
plugins:
  your-plugin-name:
    shortCut: Shift-X
    description: Your plugin description
    dangerous: false
    scopes:
      - containers # or pods, deployments, etc.
    command: bash
    background: false
    confirm: true
    args:
      - -c
      - "your command here"
```

## Additional Plugin Examples

### Custom Image with Version

```yaml
# Always use specific image version
args:
  - -c
  - "kubectl debug -it --context $CONTEXT -n=$NAMESPACE $POD --target=$NAME --image=alpine:3.19 --share-processes -- sh"
```

### Network Debug Plugin

```yaml
netdebug:
  shortCut: Shift-N
  description: Network debugging container
  dangerous: true
  scopes:
    - containers
  command: bash
  background: false
  confirm: true
  args:
    - -c
    - "kubectl debug -it --context $CONTEXT -n=$NAMESPACE $POD --target=$NAME --image=nicolaka/netshoot --share-processes -- bash"
```

## Resources

- [K9s Plugin Documentation](https://k9scli.io/topics/plugins/)
- [kubectl Debug Command](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [K9s GitHub - Plugin Examples](https://github.com/derailed/k9s/tree/master/plugins)
- [Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)

## License

This configuration is provided as-is for community use. K9s itself is licensed under Apache 2.0.
