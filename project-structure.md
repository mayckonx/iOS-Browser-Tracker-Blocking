
# Project Structure - Detailed

## For our MVP, we'll start with the following modules: 

### 🎯 Domain
Raw entities/models. It does not depend on UIKit or any persistence framework, it doesn't have implementation apart from entities.

### 📡 Network
This module provides an abstraction of the network layer as a service. Through this target, we can use all operations we need for network requests. Furthermore, as we just expose the API we can easily replace the network framework/components without affecting the Network's target consumers.

### 💾 Storage
This module provides should provide the different storages needed by the application, such as UserDefaults, CoreData, Disk Storage, and so on. The consumer of this module should easily perform those operations without knowing the details performed within the module. 

### 🧿 Core
This module provides the base and reusable classes, common UI, extensions, and protocols that can be used on the Platform module.

### 📱 Platform

The concrete implementation of the domain in a specific platform like iOS/macOS for example. 
