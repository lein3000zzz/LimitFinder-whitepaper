# LimitFinder🔎
___
### ❓ What is LimitFinder?
___
🔍[LimitFinder](https://t.me/LimitFinderNews) is a powerful tool designed to help users identify and analyze
limit orders on the cryptocurrency exchanges (can be modified to work with any exchange).
It provides a user-friendly interface in a form of (❗ **for now** ❗) a telegram bot to track order book changes, receiving alerts
for market orders based on personal user settings to make trading decisions easier and more accurate.   
This document provides a limited and simplified overview for **some of** LimitFinder's features, architecture, and technical details 
without diving into the actual codebase.

🧭 Currently *two* LimitFinder services are deployed: the main one, collecting data from the crypto exchanges, and the secondary one with a running telegram bot instance.
___
### 📌 LimitFinder info
- Project launched on February 6, 2025
- [Vladislav Severov (lein3000)](https://github.com/lein3000zzz) - owner, creator, main developer, all-in-one guy related to everything about the project
  - Responsible for everything that exists in the project, including code, infrastructure, deployment, etc.
  - That's just me, myself and I.
- [LimitFinder News Channel](https://t.me/LimitFinderNews) - official news channel for LimitFinder updates and announcements
___
### ⚙️ User Settings
The list of what a user can customize in LimitFinder 🔍:
- Choose trading pairs to monitor (*ticker whitelist feature*) (e.g., BTCUSDT, ETHUSDT, etc.)
- Choose trading pairs to ignore (*ticker blacklist feature*) (e.g., DOGEUSDT, SHIBUSDT, etc.)
- Choose trading exchanges to monitor (e.g., Binance (❗ in the currently deployed version it is Binance only, but more on that later on))
- Choose the side of the order book to monitor (e.g., 🟢BUY side, 🔴SELL side, or both)
- Set threshold volume levels for alerts (e.g., notify when a limit order exceeds a certain size)
- Set threshold estimated fill time for alerts based on the trading pair's daily volume (e.g., notify when a limit order is likely to be fulfilled within a certain time when the price reaches its level)
- Set max difference from the best bid/ask price for alerts (e.g., notify when a limit order is within a certain percentage of the current best price)
- Set minimum system detection time for alerts (e.g., avoid notifying about very short-lived orders (the params for system detection are defined by the LimitFinder itself))
- Select the frequency of updates (e.g., how often to resend the same alert for unchanged orders)
- Pause/resume monitoring at any time
___
### 📬 Alert structure
```markdown
💎 Large Order Alert! 💎

Exchange: Binance
Ticker: [ENAUSDC](https://www.binance.com/en/trade/ENA_USDC)
Side: BUY 🟢
Price: 0.3
Volume: $134356.94
Estimated Fill Time: 10.05 min
Current Price: 0.30515
Price Difference: -1.69%
System Detection Price: 0.31335
Time since system detection: 805.04 min
Time since detected for your params: 180.01 min

[]: # Links to the futures trading pages from different exchanges in a form of inline keyboard
```
___
## 📝 LimitFinder Whitepaper
___
### 📖 Creation story, PoC and initial development
- The core concept which LimitFinder is based on came from observing the cryptocurrency market and noticing how large 
limit orders can significantly impact price movements. The idea was to create a tool that could help traders identify 
these orders in real-time, allowing them to make more informed trading decisions.
- The first prototype of LimitFinder was developed in early 2025 as the Proof-of-Concept version, 
which was a simple monolithic application that could connect to a single exchange's API and monitor the order book for large limit orders.
  - The first version was built virtually accidentally with a pretty recently (at that time) released [Deno 2.0](https://deno.com/):
  one moment I was exploring Deno's features, and the next moment I found myself writing code to check websocket performance
  with Binance websocket API (what a great choice that was). From there, a pretty minimal logic was built.
  - After working on the prototype for quite some time, I realized that the idea can be used by a lot of people and 
  decided to add some basic user interface and post the project on one of the forums with no promoting or ads whatsoever.
    - It is important to note that at that time, the prototype didn't even have a persistent storage solution - all the orders 
    were stored in memory, and if the application was restarted, all the data from orderbooks was lost.
    - Moreover, the user settings were pretty much limited and only included the ability to set minimum order duration, 
    max price difference and same alert interval along with pausing/resuming the monitoring and adding blacklisted tickers
    with all the other options being decided by me as a developer.
      - All the user data was kept in MongoDB forasmuch as this PoC version needed easy flexibility for the user schema.
      As of today, the main version of LimitFinder is still using MongoDB as the primary user database since there are 
      still ideas, which I am still not sure about, that may require schema changes in the near future
      - Still, that will be changed later on when the project reaches a more mature state, and the switch to PostgreSQL is among the planned changes.
    - Meanwhile, I was still experimenting with different ideas that I could come up with to improve the prototype and make it
    more useful for users. That even included density detection of limit orders of smaller sizes that are placed very close to each other
    with a specific prefix sums logic to identify such clusters of orders and notify users about them.
    (this idea was scraped later on in the main version due to its questionable usefulness and project changes).
  - Over a few months, the prototype gained some traction, and I started receiving feedback from users, the user count 
    started to grow steadily and went beyond 150 users which made me finally decide that it was time to build a more 
    robust and scalable solution.
    - Why even bother? Well, I just love building, creating things, learning new technologies and improving my skills as a developer.
    Also, I saw a lot of potential in the project and really wanted it to be working well for as many people as possible.
    Of course, 🚨💀**unemployment**💀🚨 also played a role in dedicating more time to the project.
- By the time that was time to build a more serious version of LimitFinder, I had already devoted myself to Golang, which
also was practically the best choice for building such a system.
  - The project was still running the old version when the development of the new one started, so I no updates were added 
    to the old version during that time. One moment there were some changes made to the Binance API, and I had to stop
    it until the new version was ready (which took quite a while for a lot of reasons, including having to pause the development
    for a few weeks due to some specific circumstances).
- The new version of LimitFinder was built from scratch, with a focus on scalability, reliability, and user experience.
___
### 🚀 The main LimitFinder version
- The main version of LimitFinder features a microservices architecture, with separate services for data collection, 
processing, and user interface.
  - The data collection service connects to multiple exchanges' APIs and monitors the order books for large limit orders.
    - Currently, only Binance exchange is enabled due to the actual benefit of collecting data from additional exchanges 
    being questionable, processing power requirements growing significantly with each added exchange and currently low 
    user demand for other exchanges, but the architecture allows for easy addition of more exchanges in the future
    (in fact, it is implemented but not enabled).
  - The alerting service (telegram-bot) processes the data collected by the data collection service and sends alerts 
  to users based on their settings.
___
### 🌐 Data collection service
___
#### 🎹 The tech stack choices
- That's better to start with a tech stack used for building the main LimitFinder version, but in order to do that 
the main goals should be defined first:
  1. High performance and low latency: The system should be able to process large amounts of data in real-time, 
  with minimal delays. It is worth noting that it implies Binance websocket API usage for order book data collection, that
  sends messages every 100ms.
  2. Flexible scalability: The system should be able to handle increasing amounts of data and users as the project grows,
  but that shouldn't lead to major alterations in the codebase or architecture.
  3. Relevance: tokens are listed and delisted frequently, so the system should be able to adapt to these changes quickly
  without any manual intervention whatsoever. Moreover, daily trading volumes and market conditions change constantly, 
  so the system should be able to adjust its calculations and estimations accordingly.
  4. Flexibility: The system should be ready to changes in config and accept them "on the fly" without any manual restarts 
  or downtime.
  5. Reliability: The system should be able to recover from failures and continue operating.
  6. Maintainability: The system may not be changed for a long time, so the codebase should be clean, well-documented, 
  and easy to understand. There may be no updates, but it should be working!
  7. Monitoring: The system should have proper monitoring and alerting in place to detect issues and notify the developer.
  8. Lightweight: The system should be lightweight and not require excessive resources to run.
- Based on these goals, the following tech stack was chosen for building the data collection service:
  - Data storage: MongoDB - chosen for its flexibility, which is a big deal for this kind of project.
    - The flexibility includes both schema flexibility and easy scaling options - MongoDB cluster seemed to be a good 
    fit for the project needs (may be necessary - may be not). Plus, there would only be 1 table/collection 
    for storing detected limit orders, so no complex relational data structures were needed.
  - Remote configurations storage: HashiCorp Vault - chosen for its security features and a great key-value storage solution.
    - All the sensitive data, including API keys, database connection strings, specific tokens, etc., are stored in Vault.
    - Moreover, non-sensitive configuration options are also stored in Vault to allow for easy updates without 
    redeploying the service.
    - I have even written a basic [vault-config-manager](https://github.com/lein3000zzz/vault-config-manager) that
        helps with fetching and updating configurations from Vault in a more convenient way and also makes it automatic.
      - The service unseals the Vault, fetches the configurations on startup and also listens for changes to them, applying them 
        "on the fly" without any restarts needed. 
  - Metrics and monitoring: VictoriaMetrics (with Prometheus API) + Grafana - chosen for their powerful monitoring capabilities.
    - Why not prometheus? Well, VictoriaMetrics is known for its high performance and efficiency, which really matters for me.
    Having a prometheus-compatible solution but with better performance characteristics is a big win.
    - Grafana is used to visualize the metrics collected by Prometheus, allowing for easy monitoring and analysis of 
    the service's performance.
    - vmalert + alertmanager + alertmanager-bot are used for setting up alerts based on specific conditions in the event of issues.
    - node-exporter is used for collecting system-level metrics from the server running the service.
  - Programming Language: Golang - no comments needed, really.
    - Main libraries used:
      - [Zap Logger](https://github.com/uber-go/zap)
      - [My vault-config-manager](https://github.com/lein3000zzz/vault-config-manager)
      - [Gorilla Websocket](https://github.com/gorilla/websocket)
      - [MongoDB Golang Driver](https://github.com/mongodb/mongo-go-driver)
      - [Prometheus Golang Client](https://github.com/prometheus/client_golang)
  - Containerization: Docker, no surprises here.
  - Orchestration: Docker Compose for now, Kubernetes is planned for the future when the project grows more.
#### ⚙️ Service architecture
- It is important to define the main goals of the data collection service architecture:
  1. Modularity: The architecture should be modular, with separate components for data collection, processing, and storage.
  2. Resilience: The architecture should be designed to handle failures gracefully, with automatic recovery mechanisms in place.
  3. Real-time processing: The architecture should be designed to process data in real-time, with minimal latency.
  - With all of the above in mind, the following architecture was chosen (I have tried and scraped a lot of options actually)
- The already mentioned [vault config manager](https://github.com/lein3000zzz/vault-config-manager) is responsible for fetching
and updating configurations from HashiCorp Vault. 
  - Basically a very straightforward implementation of a config manager with hot-reloading capabilities.
  - The only funny thing is that the [vault api](https://github.com/hashicorp/vault/) sends numbers is the json.Number format, 
  which created some confusion at first. Since my config is just a wrapper for `map[string]interface{}` (`map[string]any`), I had to convert json.Number 
  to float64 manually to avoid any misunderstandings in the code.
- TickerManager:
  - TickerManager is responsible for managing the list of trading pairs (tickers) to monitor. It fetches the list of 
  trading pairs from the exchange's API and filters them based on specific system configurations (e.g. we only want
  those tickers that can be traded on the futures markets) and current ticker status. 
  This includes handling many exchanges for the most comprehensive coverage.
  - A unified exchangeInfo structure is used to effortlessly adding new exchanges in the future for ticker collection:
    ```go
    type exchangeInfo struct {
        useFuturesFlagKey     string // refers to the config key in Vault
        futuresApiUrlKey      string // refers to the config key in Vault
        futuresTickersFunc    func(url string) ([]string, error)
        useSpotTickersFlagKey string // refers to the config key in Vault
        spotApiUrlKey         string // refers to the config key in Vault
        spotTickersFunc       func(url string) ([]string, error)
    }
    ```
  - Under the hood, those functions may have different implementations for different exchanges, but the TickerManager
  doesn't care about that - it just calls the functions defined in the exchangeInfo struct.
  - TickerManager enables the service to update the list of monitored tickers automatically at regular intervals,
  ensuring that the system always has the most up-to-date information without any manual intervention.
- VolumeManager:
  - To build accurate estimated fill time calculations, VolumeManager is responsible for fetching and storing
  the 24-hour trading volumes for all monitored tickers. It fetches the volume data from the exchanges' APIs at regular
  (and really frequent, for relevance purposes) intervals and updates the internal data accordingly.
  - Similar to TickerManager, VolumeManager uses a unified structure for handling multiple exchanges which is not 
  too much different from the previous one
  - The volume is chosen based on the maximum one among exchanges for better accuracy, since usually one exchange
  has a defining factor in terms of volume for a specific trading pair.
- DataSteamManager:
  - DataStreamManager is responsible for managing the websocket connections to the exchanges' APIs. It establishes
  and maintains websocket connections for all monitored tickers, ensuring that the system receives real-time updates
  on order book changes.
    - Similar to TickerManager and VolumeManager, DataStreamManager uses a unified structure for handling multiple exchanges
    with different websocket connection implementations and parameters.
      ```go
      type exchangeDsInfo struct {
          getConnection func(currBatch []string, params []string) (*websocket.Conn, error)
          paramsKeys    []string
      }
      ```
    - As seen above, different exchanges may require different parameters for establishing websocket connections,
    so those are defined in the paramsKeys slice and fetched from the config manager accordingly.
    - Moreover, for the performance purposes, DataStreamManager batches tickers into groups.
  - The error handling and reconnection logic is implemented to ensure that the system can recover from connection failures
  automatically.
    - There are message timeouts, ping-pong and other error-detecting mechanisms in place to detect dead connections 
    and reconnect to them as soon as possible.
    - The connection is reset easily since all the data about the connections is stored in memory and can be re-established at any time.
  - Reload / restart logic is implemented - any specific connection / all the connections by exchange / all the existing connections
  can be re-established "on the fly" without restarting the whole service
    - For that purpose, all the channels are closed properly, an attempt to send data to a closed channel is handled by 
    the recover mechanism in Go, and all the goroutines are stopped gracefully before re-establishing the connections.
- OrderManager:
  - OrderManager is responsible for processing the order book data received from the exchanges' APIs. It keeps track 
  of every ticker's order book state in memory and handles anomalies such as abnormal messages, orderbook validation
  (which is actually important in the event of websocket reconnection since even that small window of missing data may lead
  to orderbook inconsistencies), etc.
  - The order book data is processed in real-time, with minimal latency, to ensure that the system can detect 
    large limit orders as soon as they are placed.
  - Currently, two modes for order book processing are implemented: strict and delta:
    - Strict mode: the volume for a specific price is replaced with the new one from the message
    - Delta mode: the volume for a specific price is increased/decreased by the amount from the message
  - Obviously, as in the case of DataStreamManager, the reload / restart logic is implemented - any specific ticker / all the tickers by exchange
  / all the existing tickers can be re-processed "on the fly" without restarting the whole service.
- Analyzer:
  - Analyzer is responsible for analyzing the order book data and detecting large limit orders based on the specific system
  settings (or more like thresholds).
  - It calculates estimated fill times based on the current order book state and 24-hour trading volumes
  fetched by VolumeManager, builds a lot of order-related data and saves it to MongoDB for storage and further processing 
  by the alerting service if the order is sufficient.
    - Analyzer is also responsible for handling the data that is not relevant anymore and cleaning it up from the database
      (e.g., orders that have been filled or canceled, orders that have been invalidated).
  - The reload / restart logic is also implemented here - any specific ticker / all the tickers by exchange
  / all the existing tickers can be re-analyzed "on the fly" without restarting the whole service.
- Metrics
  - Sounds clear, but still worth mentioning that each of the above components has its own set of metrics that are collected
  and exposed via an HTTP endpoint.
  - These metrics are then scraped by VictoriaMetrics (with Prometheus API) for monitoring and alerting purposes.
- App
  - Basically serves as a controller for all the above components, initializing them, handling their interactions,
  listening for the updates from any of them and coordinating the overall workflow of the service. All the restart requests
  (for whatever reason they are) are also handled here.
- The overall pattern used is the Pipe and Filter pattern, where data flows through a series of processing stages,
each of which performs a specific function.
  - Each ticker has its own pipeline for processing order book data, which allows for parallel processing and 
  scalability.
  - Single-worker structure showed a much better performance in comparison to worker pools and mutexes for handling
  order book data processing due to the high frequency of incoming messages and the need for real-time processing.
    - p.s there were other ideas but i dont think they are worth mentioning here, but in case you are curious, 
    feel free to ask me about them directly.
- The flow may be represented like this:

<p align="center">
    <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/ManagerFlow.png?raw=true" alt="🦍"/>
    <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/BatchFlow.png?raw=true" alt="🦍"/>
</p>

- Module dependencies may be represented like this:

<p align="center">
    <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/Dependencies.png?raw=true" alt="🦍"/>
</p>

- Some metrics (maximum recorded values for messages processed per second were about 5k, and that's just the messages incoming
for the OrderManager, not the split message parts for individual tickers):

<p align="center">
    <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/LFDashboard.png?raw=true" alt="🦍"/>
    <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/MongoDashboard.png?raw=true" alt="🦍"/>
</p>

- Overall, the architecture is designed to be modular, resilient, and scalable, allowing for easy addition of new features
and exchanges in the future as the project grows.
- Numerous performance optimizations were made throughout the codebase to ensure that the service can handle
the high frequency of incoming messages from the exchanges' APIs and process them in real-time with minimal latency.
- Funny picture from the testing phase:

<details>
    <summary>Click for a funny picture</summary>
    <p align="center">
        <img align="center" height="300" src="https://github.com/lein3000zzz/LimitFinder-whitepaper/blob/main/assets/images/readme/Atlas.jpg?raw=true" alt="🦍"/>
    </p>
</details>

___
### 🚨 Alerting telegram-bot service
___
- Currently (only for now) it is the only option to use LimitFinder for the end-users.
#### 🎹 The tech stack choices
- The alerting service is built as a telegram bot, which provides a user-friendly interface for
    receiving alerts and managing user settings.
- TypeScript with Deno runtime was chosen as the main programming language for building the alerting service.
  - There is not much to say about the tech stack choices here since Deno is a great fit for building telegram bots due to its
    simplicity, security features, and built-in support for TypeScript.
- MongoDB was chosen as the database for storing user data and settings due to its flexibility and ease of use. 
  - I am still experimenting with new parameters and features for user settings, so having a flexible schema is a necessity.
  - Still, when eventually the user settings schema is finalized, a switch to PostgreSQL is planned for better 
  reliability and performance.
- Hashicorp Vault is used for storing bot configurations and sensitive data, such as the telegram bot token and database connection strings.
#### ⚙️ Service architecture
- The alerting service is built as a monolithic application, with a single codebase handling all the functionality.
- The main components of the alerting service include:
  - Telegram Bot Interface:
    - The telegram bot interface is responsible for handling user interactions, including receiving commands,
    sending alerts, and managing user settings.
    - The bot uses inline keyboards and buttons to provide a user-friendly interface for managing settings.
  - Alert Processing:
    - The alert processing component is responsible for querying the database for all user settings and processing
    the detected limit orders accordingly.
    - It queries the database in a loop for new detected limit orders and sends alerts to users based on
    their settings. 
      - \*Only if the order's parameters satisfy the user's settings 100%
    - When the alert becomes irrelevant (e.g., the order is filled or canceled), the corresponding message is sent to 
    the user to notify them about it.
      - The order is accounted as irrelevant when:
        - The order is no longer present in the database (filled or canceled)
        - The order's params are no longer satisfying the user's settings multiplied by some lower *threshold* (currently used - 0.75)
          (e.g., the order's volume has decreased below the user's threshold multiplied by 0.75)
    - What happens when the order params are between the user's settings and the lower threshold?
      - In that case, no new alert is sent, but the existing alert is not marked as irrelevant yet, though no repeating alerts are sent for that 
      order while this is true.
      - This approach helps to avoid spamming users with alerts for orders that are still relevant but have slightly changed or 
      the orders that are on the edge of being relevant.
      - This logic is also used in the analyzer of the data collection service when deciding whether to store the order in the database or not.
  - One cycle of alert processing means 2 db queries: one for the user database to fetch all active users and their settings,
  and one for the orders database to fetch all the detected limit orders to process for each user.
    - This is achieved by making use of indexes in MongoDB for better performance and using facets to reduce the number of queries needed.
- Once again, numerous performance optimizations were made throughout the codebase to ensure that the service can handle
the high frequency of incoming detected orders from the data collection service and process them in real-time with minimal latency.
- Still, the telegram bot service has a bottleneck in the form of telegram API limitations, which include not instantly delivering messages
and rate limits for sending messages.
