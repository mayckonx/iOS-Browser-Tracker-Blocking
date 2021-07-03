
# Project Structure - Tech Stack

## Below, we have the initial technologies that will be applied to this project, considering the MVP's scope. 

### Native libraries

### Swift 
For native development and for UI components, for a few special cases like Widgets, we will use SwiftUI. 

### Combine
The native option provided by Apple to support a unified declarative API for process values over time. We will Combine to dealing with delegates, notifications, timers, completion blocks. By using Combine, we have common streams for different ways to handle async tasks. As in this project, we will support iOS13+ Combine fits well for this case.  

### UserDefaults

To store user's preferences data such as to enable/disable blocker tracking, etc.

---
  
### External Libraries

### Moya
- [Moya](https://github.com/Moya/Moya) is a well-maintained network library that provides a comprehensive and simple abstraction of the callings for Alamofire to perform network requests of any kind.

