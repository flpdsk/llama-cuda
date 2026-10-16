# llama-cuda

Reproducible CUDA build of [llama.cpp](https://github.com/ggml-org/llama.cpp)
using Nix flakes and GitHub Actions.

## What is built

The repository exposes:

```bash
.#llama-cpp-cuda

The package is based on pkgs.llama-cpp with CUDA enabled:

pkgs.llama-cpp.override {
  cudaSupport = true;
}

This repository builds the llama.cpp binaries only. Model files such as GGUF
weights are not included.

## Build locally

Requires Nix with flakes enabled:

nix build .#llama-cpp-cuda --print-build-logs

Check the result:

./result/bin/llama-server --version

To avoid creating result in the repository:

nix build .#llama-cpp-cuda \
  --print-build-logs \
  --out-link /tmp/llama-cpp-cuda

## GitHub Actions

The repository contains a workflow:

.github/workflows/build-llama-cuda.yml

It builds the package on:

- changes to flake.nix;
- changes to flake.lock;
- manual workflow execution.

The resulting Nix store paths are uploaded to the public Cachix cache:

flpdsk-llama

The workflow requires the following GitHub repository secret:

CACHIX_AUTH_TOKEN

The token should have write access only to the flpdsk-llama cache.

## Cachix

To use the cache on NixOS, add it to configuration.nix:

nix.settings = {
  substituters = [
    "https://cache.nixos.org"
    "https://flpdsk-llama.cachix.org"
  ];

  trusted-public-keys = [
    "cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY="
    "flpdsk-llama.cachix.org-1:3PL+05hHvRFQoZD3jiIMU7f5dDg/dvGCVpTULczU+tQ="
  ];
};

Then apply the configuration:

sudo nixos-rebuild switch --flake /etc/nixos#nixos

If the required package is available in Cachix, Nix will download it instead
of compiling it locally.

## Updating dependencies

The repository uses flake.lock to pin the exact nixpkgs revision.

Do not update dependencies automatically on every build. Update them manually
when needed:

nix flake update --update-input nixpkgs

After updating flake.lock, run the GitHub Actions workflow and verify that
the CUDA build succeeds.

## Runtime requirements

The binary requires:

- an NVIDIA GPU;
- a compatible NVIDIA driver on the target system;
- a CUDA-compatible Linux environment.

CUDA support is compiled into the binary, but the GitHub Actions runner does
not need a GPU for compilation.

## Security notes

- Do not commit Cachix tokens or other secrets.
- Store tokens only in GitHub repository secrets.
- Review changes to flake.nix and flake.lock before updating dependencies.
- The repository contains build configuration, not model weights.
