## Install required CLIs

- Install the [Github CLI](https://github.com/cli/cli) (version 1.14.0+)
- Install the OpenShift CLI `oc` (version 4.7 or 4.8)

    - Log into your OCP cluster, substituting the `--token` and `--server` parameters with your values:

        ```bash
        oc login --token=<token> --server=<server>
        ```

        If you are unsure of these values, click your user ID in the OpenShift web console and select `Copy login command`.

- Install the [kubeseal CLI](https://github.com/bitnami-labs/sealed-secrets#homebrew) 
