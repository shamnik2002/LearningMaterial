# Simple Crypto Currency Project Using Claude

Displays a set of predefined crypto currencies. Uses Coinbase Exchange API to fetch the real time crypto currencies data like price, volume etc and displays it.
App has 2 views, a list view and a detail view to allow users to view basic details of the cyrpto currency.

## Overall Experience
- Great experience
- With enough context in the project level Claude.md file, the prompts, some prior contexts of my other projects, Claude was able to create a project within 15 mins
- I built the same project without the context and details and it didn't follow any architecture or coding standards. So the better context and predefined specs you give the better results you will see.
- Same project wihout the fancy UI took me 12 - 15 hours, although it includes me reading coinbase APIs and learning websockets. Still very impressive to see claude doing this within 15 mins.
- Code review took ~3 hours since I was making notes as well
- So dev (12 - 15 hours) vs Claude (3 hours 15 mins + time it took to design the system and build prompts ~ 1 hour), very impressive.
- learnings/ next steps: I could improve my prompts to consume less token and be more clear upfront.

## Projects

### Project without context/specifications

### Project with context + specifications


## System Requirements

- Built using Xcode 26.3 + Claude
- Built entirely via Claude. no manual code.
- Dev inputs include defining initial project level Claude.md file, prompts, code review and additional prompts to address issues based on code review feedback.


## Prior Claude set up

- Install latest Xcode 26.3
- Get a pro claude subscription
- Set up claude in xcode settings

## Project level Claude.md

- Specify the basic dev expectations here
- Like platform, min OS dependency, UI framework, dependency management, etc
- Define project structure like where to put certain files
- Coding standards like MVVM, SOLID, dependency injection, protocols, any specifics like create separate files for low level work like network/web socket service, a ticker service that view models can talk to etc
- Swift style to make sure we modularize views instead of a single view holding everything
- Navigation pattern , I used coordinator here and gave an example
- Error handling to make sure readable errors are thrown, also gave an example here

## Initial Prompt

This is more like a PRD, with some additional context on how to build. Mine was pretty simple. You could also provide screenshots for UI, but I didn't.
This is based on what I did when I built the project. So thinking of a basic system design and providing that helps. 

```
Build Real time Crypto currency viewer App with Coinbase API

API: Use coinbase exchange API https://docs.cdp.coinbase.com/api-reference/exchange-api/rest-api/introduction

UI:
Show a list of predefined (10 popular) crypto currency
The price should update real time
Should show change since last open

Network Layer
Use websockets for real time data
Handle disconnect
Should handle disconnecting web sockets when app goes to background, resume when app comes to foreground

Data/Caching Layer

In memory cache to hold data in case app comes to foreground
Simple file as persistent store to save data

Service layer
View Model relies on this to get data
Checks if data is available in cache/persistent store to display stale data in case device is offline
Fetches from network 
Handles throttling to reduce UI overhead

```

## Review

### What Claude did great

UI
- UI was clean, intuitive and very well done
- Used SFSymbols for change since last open

Persistent Store
- Used actor without having to specify
- Enum for Errors
- protocol for store methods

Cache
- Protocol for cache methods
- Used actor along with generics/type aliases

Data models
- Clean, well defined data models
- Added all formatting methods in an extension

WebsocketMessage Models
- Again, clean, well defined structs for any message sent or received from web socket

CoinbaseAPIClient
- This was for rest APi, in case sockets didn't get data soon enough
- This handled the discrepancy where socket returned ticker product_id vs rest api didn't. But claude caught that and saved product id before writing to store

CoinbaseWebsocket
- Made it an actor, since there was shared state + need to serialize requests

AppLifeCycleManager
- Handled stopping/restarting socket on app state


## What Claude required fixing after code review

UI
- In the list view the row was wrapped in a button, and tapping on the white space between crypto name and price did nothing.
  
Persistent Store
- Was saving the file on caches directory which could be removed by iOS at any point
- Added unused unnecessary methods to both protocol and concrete class

CoinbaseAPIClient
- Made this an actor. Didn't have any shared state/data. And no requirement of serializing the requests.
- Created fetch for generic data fetch requests, handled all errors here. Also create fetchTicker which didn't reuse the fetch method, instead redid everything here, missed handling all errors.
- Fetch and fetchTicker did everything, creating a request, fetch data, parsing etc. No clear separation.

CoinBaseWebsocket
- Used private _<var name> and public computed property over private(set)
- Added unnecessary code for extra pingTask to handle socket failure. This was already being done within the receiveMessage for the ticker websocket.
- Error hanlding could be improved
- Created unused WebSocketMessageHandler protocol

TickerService
- On startStreaming, always read data from persistent store before checking if cache was empty. unnecessary file read if data was already available in cache.


## Prompts to fix issues identified in code review

1.
```
@``TickerRowView`` tapping in the blank space between the ticker name and price does nothing. To go to detail view I have to tap on the name or the price. Can you fix this ?
```
NOTE: not sure if it figured out on it’s own, or kept track of my testing with my fix, removing code and then asking. But regardless, the issue was fixed.


2. 
```
@``TickerPersistentStore``  has unused functions save/load/remove , can you remove them from both the protocol and the concrete class
```

3.
```
@``TickerPersistentStore``  saves data in caches directory, change that to application support directory so iOS won't get rid of it
```

4.
```
@``CoinbaseAPIClient`` why does this need to be an actor ? there is no shared data
```

5.
```
CoinbaseAPIclient creates request, fetches data and parses it. Let's separate request creation out.1. create a requestProtocol e.g protocol RequestProtocol {
    var host: String {get}
    var path: String {get}
    var scheme: Scheme {get}
    var queryParams: [String: String]? {get}
    
    func buildRequest() throws -> URLRequest
} 2. create a concrete struct that confroms to this and builds the request to fetch ticker
```
Did a great job of creating necessary files and adding them in expected folder structure. cleaned up the API client file as well to remove the redundant code and improved validation

6.
```
@``CoinbaseWebSocket`` use swift private(set) over _<variable> and public computed property  for _connectionState
```

7.  Unnecessary back and forth here with claude on remove pingTask from CoinbaseWebSocket. But I'm glad it doesn't just agree with me all time though.

```
@``CoinbaseWebSocket`` is the pingTask used, if not should remove it
```
<Claude response> It's used to keep the WebSocket connection alive by sending periodic pings and to properly cancel the ping loop on disconnect. It should be kept.

```
why is it needed when there is receiveMessages 
```
<Claude response> You're right. The start​Receiving() method is unnecessary - it just wraps receive​Messages() in a Task. We can call receive​Messages() directly in a Task from connect().

```
@``CoinbaseWebSocket``  why is pingTask needed when we have webSocketTask and it keeps the socket alive until we recieve error from server
```
<Claude response> You're right. The WebSocket connection will stay alive as long as we're receiving messages, and if the server disconnects, receive​Messages() will catch the error and handle it. The ping mechanism is redundant.

8.
```
@WebSocketMessageHandler is this protocol used, if not remove it
```

9.
```
@``TickerService``  loadCachedData always reads from persistentStore instead of checking if there is already something in cache. Should only go to persistent store if cache is empty
```
