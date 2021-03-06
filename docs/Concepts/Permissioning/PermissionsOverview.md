# Permissioning

GoQuorum supports two network permissioning models.

* [Basic network permissioning](BasicNetworkPermissions.md): Controls which nodes can connect to a given node and also to which nodes the given node can dial out to.
* [Enhanced network permissioning](Enhanced/EnhancedPermissionsOverview.md): Designed for enterprise-level needs by having a **smart contract-based permissioning model**. This allows for significant flexibility to manage nodes, accounts, and account-level access controls. Two versions of enhanced network permissioning are available::

    * **v1:** This is the original permissions model. When using this model, the permissions rules are applied only at the time of transactions entry with respect to the permissions data stored in memory of the node.
    * **v2:** This enhances the original model. When using this model, the permissions rules are applied both at the time of transaction entry and block minting with respect to the data stored in permissions contract.
