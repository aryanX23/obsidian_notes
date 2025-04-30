# Elastic Block Storage - 

- An Amazon EBS volume is a durable, block-level storage device that you can attach to one or maore instances.
- After a volume is attached to an instance, you can use it like any other physical hard-drive.
- EBS volumes are flexible.
- An EBS volume and the instance to which it attaches must be in the same availability zone.
![[Pasted image 20230615194231.png|700]]

--- 

## Amazon EBS Volume Types - 

Amazon EBS provides the following volume types, which differ in performance characteristics and price.

* SSD-backed volumes optimized for transactional workloads involving frequent read/write operations
with small I/O size, where the dominant performance attribute is IOPS

* HDD-backed volumes optimized for large streaming workloads where throughput (measured in MiB/s)
is a better performance measure than IOPS

- Magnetic (standard) - 
	Magnetic volumes are backed by magnetic drives and are suited for workloads where data is accessed
	infrequently, and scenarios where low-cost storage for small volume sizes is important. These volumes
	deliver approximately 100 IOPS on average, with burst capability of up to hundreds of IOPS, and they can
	range in size from 1 GiB to 1 TiB.

`Note: Magnetic is a Previous Generation Volume. For new applications, AWS recommend using one of the newer volume types.`

---

## Snapshots — Root Volumes and Data Volumes - 

* Can back up the data on your Amazon EBS volumes to Amazon S3
* Snapshots are incremental backups, which means that only the blocks on the device that have
changed after your most recent snapshot are saved to minimizes the time

### Root Volume - 

* When you launch an instance, the root device volume contains the image used to boot the instance.
When we introduced [[Amazon EC2(Amazon Elastic Compute Cloud)]], all AMIs were backed by Amazon EC2 instance store, which means
the root device for an instance launched from the AMI is an instance store volume created from a
template stored in Amazon S3.

### Data Volume - 

* An Amazon EBS volume is a durable, block-level storage device that you can attach to a
single EC2 instance.

* Can use EBS volumes as primary storage for data that requires frequent updates, such as the system
drive for an instance or storage for a database application.

### Instance Store Volumes - 

- An instance store provides temporary block-level storage for your instance                        
running, but this data is deleted when the instance is terminated or if it fails
- This storage is located on disks that are physically attached to the host computer
- Instance store is ideal for temporary storage of information that changes frequently,
such as buffers, caches, scratch data, and other temporary content.

![[Pasted image 20230615200516.png|300]]

--- 
## Benefits of using EBS volumes - 

EBS volumes provide benefits that are not provided by instance store volumes.

### Topics - 

- [Data availability](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#availability-benefit)
- [Data persistence](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#persistence-benefit)
- [Data encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#encryption-benefit)
- [Data security](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#security-benefit)
- [Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#backup-benefit)
- [Flexibility](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html#flexibility-benefit)
---

## Types of Elastic Block Storage in Detail - 

### Solid state drive (SSD) volumes -

SSD-backed volumes are optimized for transactional workloads involving frequent read/write operations with small I/O size, where the dominant performance attribute is IOPS. SSD-backed volume types include **General Purpose SSD** and **Provisioned IOPS SSD** . The following is a summary of the use cases and characteristics of SSD-backed volumes.

![[Pasted image 20230615201307.png|1400]]

`The throughput limit is between 128 MiB/s and 250 MiB/s, depending on the volume size.Volumes created before **December 3, 2018** that have not been modified since creation might not reach full performance unless you [modify the volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modify-volume.html).`

- To achieve maximum throughput of 1,000 MiB/s, the volume must be provisioned with 64,000 IOPS and it must be attached to an [instance built on the Nitro System](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances). `io1` volumes created before **December 6, 2017** and that have not been modified since creation, might not reach full performance unless you [modify the volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modify-volume.html).

For more information about the SSD-backed volume types, see the following:
- [General Purpose SSD volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose.html)
- [Provisioned IOPS SSD volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/provisioned-iops.html)

### Hard disk drive (HDD) volumes - 

HDD-backed volumes are optimized for large streaming workloads where the dominant performance attribute is throughput. HDD volume types include **Throughput Optimized HDD** and **Cold HDD**. The following is a summary of the use cases and characteristics of HDD-backed volumes.

![[Pasted image 20230615201625.png|1400]]


### Magnetic (`standard`) volumes - 

Magnetic (`standard`) volumes are previous generation volumes that are backed by magnetic drives. They are suited for workloads with small datasets where data is accessed infrequently and performance is not of primary importance. These volumes deliver approximately 100 IOPS on average, with burst capability of up to hundreds of IOPS, and they can range in size from 1 GiB to 1 TiB.

The following table describes previous-generation EBS volume types - 

![[Pasted image 20230615201801.png|1000]]

----
