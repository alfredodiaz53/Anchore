# To upgrade the Anchore Package

Check the [upstream release notes](https://docs.anchore.com/current/docs/releasenotes/) and the [helm chart upgrade notes](https://github.com/anchore/anchore-charts/tree/main/stable/anchore-engine#upgrading-from-previous-chart-versions).

### Upgrading

Find the latest anchore-engine chart version that corresponds with the Anchore Enterprise version identified by Renovate.

Update the chart with KPT
```shell
kpt pkg update chart@anchore-engine-${chart.version} --strategy alpha-git-patch
```

### Modifications made to upstream
Review the list of [Big Bang Changes](https://repo1.dso.mil/big-bang/product/packages/anchore-enterprise/-/blob/main/docs/BBCHANGES.md) to this chart and ensure they weren't overwritten in the update.

# Testing New Anchore Version

### Deploy Anchore as part of Big Bang

- Obtain the Big Bang dev Anchore enterprise license by following the below instructions:
  - Clone the dogfood repo if you have not already, from https://repo1.dso.mil/big-bang/team/deployments/bigbang.git
  - Run `sops -d bigbang/prod/environment-bb-secret.enc.yaml | yq '.stringData."values.yaml"' | yq '.addons.anchore.enterprise.licenseYaml'` to get the full license contents.
  - Add the full output from that command under `licenseYaml` in your override values (shown below), making sure that indentation is properly preserved 

`overrides/anchore.yaml`
```
addons:
  anchore:
    enabled: true
    git:
      tag: Null
      branch: "renovate/anchore"
    adminPassword: "foobar"
    enterprise:
      enabled: true
      licenseYaml: |
            $LICENSE_CONTENT
    sso:
      enabled: true
      client_id: "platform1_a8604cc9-f5e9-4656-802d-d05624370245_bb8-anchore"
    values:
      anchoreAnalyzer:
        replicaCount: 2
```
- Deploy Big Bang and Anchore to dev environment
```
helm upgrade -i bigbang ./bigbang/chart --create-namespace -n bigbang -f ./bigbang/chart/ingress-certs.yaml -f ./overrides/registry-values.yaml -f ./overrides/anchore.yaml
```
NOTE: You may disable `kiali`, `kyverno`, `promtail`, `loki`, `neuvector`, `tempo`, and/or `monitoring` in the deployment, if desired, as they are not required for testing.

- [ ] Visit `https://anchore.bigbang.dev`
- [ ] Confirm ability to login with Keycloak (SSO)
- [ ] Provide your P1 SSO credentials and confirm successful login.
- [ ] Logout, then log in with the username and password. Login as 'admin' using the password specified in the overrides file
- [ ] Confirm the intended version and review the release notes by selecting the version number in the upper left. Ensure there are no important or breaking changes that need to be addressed
- [ ] Navigate to `Images` and select `Analyze Tag` to add a new tag for analysis.
- [ ] Populate the fields with a registry/image/tag of your choosing, or by using the example information.
- [ ] Allow several minutes for the analysis to complete.
- [ ] Select the repository name of your new tag, confirm `Status` is `Analyzed`
- [ ] Select the tag SHA and confirm `Metadata`, `Policy Compliance`, and `Action Workbench` have all been Analyzed. (`Vulnerabilities` will be marked unsuccessful, with a red 'X,' this is expected.)
