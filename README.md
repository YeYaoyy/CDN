- High-level approach
    - To deploy the CDN, use ./deployCDN -p <port> -o <origin> -n <name> -u <username> -i <keyfile>, this will deploy everything needed including databases for DNS server and all replicas
    - To activate the CDN, use ./runCDN -p <port> -o <origin> -n <name> -u <username> -i <keyfile>, this will turn on subprocesses on each server;
    - To stop the CDN, use ./stopCDN -p <port> -o <origin> -n <name> -u <username> -i <keyfile>, this will kill all relevant processes, but the files still remain on the disk.

- Implementation:
    - DNS Server
      - To implement the DNS server, we use the GeoIP database to help locate client request physical address.
Since the database file is too big to be accepted by gradescope, during the deployment stage, we use python.requests and python.zipfile libraries to
download database and unzip the database to extract the IPv4.csv file. To make good use of the csv file, we use python.sqlite3 library to build a database
named geoip.db to help search for physical longitude and latitude. Because the raw csv file only contains subnet, so we need to find the max host address and min host address of each subnet and store both of them in the
geoip.db file. Since all ip addresses are stored as numeric, database index can be set up to accelerate the search and time complexity can be reduced to log(N).
The database technique is a good strategy since we don't need to spend time waiting for RTT from geoip server. Besides, we also use mul-threaded server and python.dnslib to help 
pack DNS request dealing with multiple client requests at the same time.
    - HTTP Server
      - Use requests library to send GET request to origin server to get contents that are not in replica cache, and handle unicode error.
      - Use BaseHTTPRequestHandler to handle GET request from clients by overriding do_GET() function.
      - Use threading to handle request from clients to avoid program crushing.
      - Use zlib to compress size of file to save more files in restricted cache.
      - Save pages as file format to reduce modifying file time.
      - Use LRU algorithm to implement cache management strategy and rank each file's priority.
      - Control cache below 17MB to avoid out memory.
      - Use dictionary to save cache data previously to save searching and operating files time, which will be O(1).
      - Use list to provide rank of each file's priority.
      - Use unquote() to avoid unicode mistake.
- Design Decisions:
    - Save top 170 pages ordered by viewing frequency in cache.
    - To save more pages, adopting compress string.
    - Use LRU algorithm to update cache, if a new page is requested, it will be save in cache and remove the page of the lowest viewing frequency.

- Challenges:
    - for dns server, it requires math knowledge to calculate distance between two geological points and the max, min hosts of a subnet.
    - Some pages include hidden url in their response body and requests library will call another thread automatically. These hidden url include special character affecting writing file because they need use subdirectory which often cause filenotexists error.
    - Url send by clients includes specical character, which need to be transfered to avoid not searching it in cache.
    - Need to find suitable method to compress string so that it will not consume more time.

- Future enhancements:
    - for dns server, we could probably use scamper to do active probes.
    - we could build a protocol between dns server and httpservers so dns server can intelligently know the contents cached on replicas. 
    - connect with DNS to make the cache strategy more smart and DNS can response the nearst replica who owns the page in cache.
    - find more effective method to rank each page's priority
- Group details:
    - Xuran Feng: responsible for dns server
    - Ye Yao: responsible for http server