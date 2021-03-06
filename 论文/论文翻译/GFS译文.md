# The Google File System

## 摘要（ABSTRACT）

我们已经设计并实现了Google File System，这是一个可扩展的分布式文件系统，用于大型分布式数据密集型应用程序。 它可以在廉价的日用硬件上运行时提供容错功能，并且可以为大量客户端提供高聚合性能。

​	在实现与以前的分布式文件系统相同的许多目标的同时，我们的设计是由对我们当前和预期的应用程序工作负载和技术环境的观察所推动的，这反映出与某些早期文件系统假设的明显背离。 这导致我们重新审视传统选择，并探索根本不同的设计要点。

​	文件系统已成功满足了我们的存储需求，已在Google内部广泛部署为存储平台，用于生成和处理我们的服务使用的数据以及需要大量数据集的研发工作。 迄今为止最大的集群可在一千多台计算机上的数千个磁盘上提供数百TB的存储，并且数百个客户端可以同时访问。

​	在本文中，我们介绍了旨在支持分布式应用程序的文件系统接口扩展，讨论了我们设计的许多方面，并报告了微基准测试和实际使用情况的测量结果。

## 类别和主题描述符（Categories and Subject Descriptors）

D [4]: 3—Distributed file systems 

## 一般条款（General Terms）

Design, reliability, performance, measurement 

## 关键词（Keywords）

Fault tolerance, scalability, data storage, clustered storage  



允许为以下目的制作全部或部分本作品的数字或纸质副本，如果不为牟利或商业利益而制作或分发副本，并且副本载有此声明和第一页的完整引用，则可免费提供个人或教室使用。 若要进行其他复制，重新发布，在服务器上发布或重新分发到列表，则需要事先获得特定的许可和/或费用。

SOSP’03, October 19–22, 2003, Bolton Landing, New York, USA.
Copyright 2003 ACM 1-58113-757-5/03/0010 ...$5.00.  

## 1. 绪论（INTRODUCTION）

​	我们已经设计并实施了Google文件系统（GFS），以满足Google数据处理需求快速增长的需求。 GFS与以前的分布式文件系统具有许多相同的目标，例如性能，可伸缩性，可靠性和可用性。 但是，其设计是由对我们当前和预期的应用程序工作负载和技术环境的主要观察驱动的，这反映出与某些早期文件系统设计假设存在明显差异。 我们重新审视了传统选择，并探索了设计空间中的根本不同点。

​	首先，组件故障是正常现象，而不是例外情况。 文件系统由数百个甚至数千个廉价的商品部件构建的存储机器组成，并且可被相当数量的客户端计算机访问。 组件的数量和质量实际上也确定了部分组件一定会在某些时间无法运行，另一些则无法从其当前故障中恢复。 我们已经看到了由应用程序错误，操作系统错误，人为错误以及磁盘，内存，连接器，网络和电源故障引起的问题。 因此，持续监视，错误检测，容错和自动恢复必须是系统不可或缺的。

​	其次，按照传统标准来说文件很大，GB规模的文件是常见的。每个文件通常包含许多应用程序对象，例如Web文档。 当我们定期处理包含数十亿个对象的许多以TB级的快速增长的数据集时，即使文件系统可以支持数十亿个大约KB大小的文件，也难以管理。结果，必须重新考虑设计假设和参数，例如I/O操作和块大小。

​	第三，大多数文件是通过附加新数据而不是覆盖现有数据来进行突变的。文件内的随机写入实际上是不存在的。写入后，仅读取文件，并且通常只能顺序读取，各种数据共享这些特征。有些可能会构成大型存储库，数据分析程序会扫描这些存储库。一些可能是正在运行的应用程序连续生成的数据流，有些可能是档案数据，某些结果可能是在一台机器上产生的中间结果，而在另一台机器上同时产生或稍后产生。 鉴于这种对大型文件的访问模式，附加成为性能优化和原子性保证的重点，而在客户端中缓存数据块却失去了吸引力。

​	第四，共同设计应用程序和文件系统API可以提高灵活性，从而使整个系统受益。例如，我们放松了GFS的一致性模型，大大简化了文件系统，而又不给应用程序带来繁重的负担。我们还引入了原子附加操作，以便多个客户端可以并发附加到文件，而无需在它们之间进行额外的同步。 这些将在本文后面详细讨论。

​	当前部署了多个GFS群集以用于不同的目的。 最大的服务器具有1000多个存储节点，超过300 TB的磁盘存储，并且连续不断地被数百台不同计算机上的客户端访问。

## 2. 设计概述（DESIGN OVERVIEW）

### 2.1 假设条件（Assumptions）

​	在设计满足我们需求的文件系统时，我们以既带来挑战又带来机遇的假设为指导。我们之前提到了一些关键的观察，现在更详细地阐述了我们的假设。

* 该系统由许多经常发生故障的廉价商品组件组成，它必须不断地自我监控，并定期检测，容忍，并及时从组件故障中恢复。
* 系统存储少量的大文件。 我们期望有几百万个文件，每个文件的大小通常为100 MB或更大。 GB级别的文件也是常见的情况，应进行有效管理。 必须支持小文件，但我们不需要对其进行优化。
* 工作负载主要包括两种读取：大流读取和小随机读取。 在大型流读取中，单个操作通常读取数百KB，更常见的是1MB或更多。来自同一客户端的成功操作通常会读取文件的连续区域。 小型随机读取通常以任意偏移量读取几个KB。注重性能的应用程序通常会对小型读取进行批处理和排序，以稳定地通过文件而不是来回移动。
* 工作负载还具有许多大的顺序写入，这些写入将数据追加到文件中。 典型的操作大小类似于读取的大小，一旦写入很少再修改文件。 支持对文件中任意位置的小写操作，但不一定要有效。
* 系统必须为同时附加到同一文件的多个客户端有效地实现定义良好的语义。 我们的文件通常用作生产者-消费者队列或用于多种合并。每台计算机上运行一个生产者的数百个生产者将同时添加到文件中。 具有最小同步开销的原子性是必不可少的。该文件可能会在以后读取，或者消费者可能正在同时读取文件。
* 高持续带宽比低延迟更重要，我们的大多数目标应用程序都非常重视以高速率处理大量数据，而很少有对单个读取或写入有严格响应时间要求的应用程序。

### 2.2 接口（Interface）

​	GFS提供了一个熟悉的文件系统接口，尽管它没有实现诸如POSIX之类的标准API。 文件在目录中按层次结构组织，并由路径名标识。 我们支持创建，删除，打开，关闭，读取和写入文件的常规操作。

​	此外，GFS具有快照和记录附加操作。 快照可以低成本创建文件或目录树的副本。 记录追加允许多个客户端同时将数据追加到同一个文件中，同时保证每个客户端的追加的原子性。 这对于实现多路合并结果和生产者/消费者队列非常有用，许多客户端可以在不附加锁定的情况下将其同时附加到该队列。我们发现这些文件类型对于构建大型分布式应用程序非常重要。快照和记录追加分别在3.4和3.3节中进一步讨论。

### 2.3架构（Architecture）

​	一个GFS集群由一个主服务器和多个块服务器组成，并且可以由多个客户端访问，如图1所示。它们中的每一个通常都是运行用户级服务器进程的商用Linux计算机。只要机器资源允许，并且可以运行不稳定的应用程序代码而导致较低的可靠性，就可以在同一台机器上同时运行chunkserver和客户端。