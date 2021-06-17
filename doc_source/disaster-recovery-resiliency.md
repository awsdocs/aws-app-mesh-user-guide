# Resilience in AWS App Mesh<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around AWS Regions and Availability Zones\. AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between Availability Zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\.

App Mesh runs its control plane instances across multiple Availability Zones to ensure high availability\. App Mesh automatically detects and replaces unhealthy control plane instances, and it provides automated version upgrades and patching for them\.

## Disaster recovery in AWS App Mesh<a name="disaster-recovery"></a>

The App Mesh service manages backups of customer data\. There is nothing that you need to do to manage backups\. The backed\-up data is encrypted\.