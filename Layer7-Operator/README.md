# Layer7 Operator
The Layer7 Gateway Operator, built using the Operator SDK covers all aspects of deploying, maintaining and upgrading API Gateways in Kubernetes.

[You can find the Layer7 Operator here](https://github.com/gazza7205/layer7-operator)

### About
The Operator is currently in an Alpha state and therefore does not currently carry a support/maintenance statement. Please check out the Gateway Helm Chart for supported container Gateway Deployment options

The initial release gives basic coverage to instantiating one or more Operator managed Gateway Deployments with or without an external MySQL Database.

The Layer7 Operator is restricted to manage the namespace it's deployed in by default. There is also a cluster-wide option available where the operator can watch all or multiple namespaces.