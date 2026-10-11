# Minerals Platform organization configuration

Shared GitHub configuration and reusable CI/CD workflows for trusted private
repositories in the `minerals-platform` organization.

## Reusable container workflow

Service repositories call `.github/workflows/container-ci.yml`. Pull requests
build the image on the isolated organization runner. Commits to `main` rebuild
and publish the immutable `${GITHUB_SHA}` tag from the same trusted runner, then
optionally invoke a Coolify deploy webhook. The runner has no direct access to
the production Docker roots or installed runtime configuration.

Required organization/repository configuration:

- variable `CONTAINER_REGISTRY` (for example `registry.example.com`)
- secrets `REGISTRY_USERNAME` and `REGISTRY_PASSWORD`
- variables `COOLIFY_URL` and `COOLIFY_APPLICATION_UUID` for each Docker Image application
- optional secret `COOLIFY_TOKEN` with application update/deploy permissions
- a self-hosted runner with labels `linux`, `x64`, and `minerals-ci`
- a protected `production` environment in every deployable repository

The runner must be assigned only to a runner group that allows selected trusted
private repositories. Do not enable it for public repositories or fork PRs.

When Coolify variables and token are present, the workflow first changes the
Docker Image application's registry tag to the full Git SHA through the Coolify
API, then starts a deployment. This preserves immutable production identity.
