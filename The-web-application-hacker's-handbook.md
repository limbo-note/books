# http://mdsec.net/wahh/ #
# 1. Web Application (In)secutiry #
## Web Application Secutiry ##
some common categories of vulnerability:
- Broken authentication
- Broken access controls
- SQL injection
- Cross-site scripting
- Information leakage
- Cross-site request forgery

## Core Security Problem: Arbitrary input ##
manifests in ways:
- interfere with request parameters, cookies, HTTP headers
- send requests in any sequence, submit unexpected parameters
- not only use a browser

# 2. Core Defense Mechanisms #
- handling user access
- handling user input
- handling attackers
- enabling admins to monitor activities

## Handling user access(a trio of mechanisms, interdependent, weakest) ##
- **Authentication**
login, certificates, smartcards, tokens
- **Session management**
cookies 
- **Access control**

## Handling user input ##
- Varieties of input
registration, blog, eg.
- how to handle input
 1. reject known bad: **blacklist.** 
 but many any can be bypassed:
 SELECT -> SeLeCt
 or 1=1-- -> or 2=2--
 alert('xss') -> prompt('xss')
 ...
 2. accpet known good: **whitelist.**
 3. sanitization. accpet input that can be bad, sanitize it.
 4. safe data handling. 
 SQL, parameterized queries.
 5. semantic checks
- Boundary validation
an example:
 1. The application receives the user’s login details. The form handler validates that each item of input contains only permitted characters, is within a specific length limit, and does not contain any known attack signatures.
 2. The application performs a SQL query to verify the user’s credentials.To prevent SQL injection attacks, any characters within the user input that may be used to attack the database are escaped before the query is constructed.
 3. If the login succeeds, the application passes certain data from the user’s profile to a SOAP service to retrieve further information about her account.To prevent SOAP injection attacks, any XML metacharacters within the user’s profi le data are suitably encoded.
 4. The application displays the user’s account information back to the user’s browser. To prevent cross-site scripting attacks, the application HTML-encodes any user-supplied data that is embedded into the returned page.
- Multistep Validation and Canonicalization

## Handling Attackers ##
- Handling errors
never return system-generated messages or debug info to user
- Maintaining audit logs
- Alerting admins
- Reacting to attack


# 3. Web Application Technologies #
## The HTTP Protocol ##
Http is **connectionless**,although it's based on TCP,each exchange of request and response may use a different TCP connection.

- HTTP Requests
 1. one or more headers
 2. separate line
 3. followed by a mandatory blank line
 4. followed by a optional body
 5. the first line consists of 3 items:
   - a verb indicating HTTP method
   - requested URL
   - HTTP version, in 1.1, *Host* is mandatory
 6. *Referer*: the request originated from
 7. *User-agent*: info about the browser or client software
 8. *Host*: hostname
...
...so on
- HTTP Responses
 1. first line:
   - HTTP version
   - status code indicating the result
   - reason describing the status
 2. *Server*: the server software, may not accurate
 3. *Set-Cookie*: further cookie for next request
 4. *Pragma*: browser store the response in cache
 5. *Content-Type*: the body type
 6. *Content-length*: the body length
...
...so on
- HTTP Methods
*Get*: only send parameter in URL query string
*Post*: can send parameter both in URL query string and message body
other methods: *HEAD, TRACE, OPTIONS, PUT*
- URLs
*protocol://hostname[:port]/[path/]file[?param=value]*
- REST
*http://wahh-app.com/search?make=ford&model=pinto*
↓
*http://wahh-app.com/search/ford/pinto*
- HTTP Headers
**PDF81**: general headers, request headers, response headers
- Cookies
*Set-Cookie, Cookie, expires, domain, path, secure, HttpOnly*
- Status Codes
**PDF84**
- HTTPS
HTTP+SSL
- HTTP Proxies
difference between HTTP and HTTPS
- HTTP Authenticaiton
Basic, NTLM, Digest

## Web Functionality ##
- server-side functionality
 generate contents dynamically accroding to parameter, parameter can be sent in ways:
 1. URL query string
 2. REST-style URLs
 3. HTTP cookies
 4. body of POST requests

 a wide range of technologies(**PDF89-93**):
 1. scripting languages: PHP, VBScript, Perl
 2. web application platform: ASP.NET, Java
 3. web server: Apache, IIS, Netscape Enterprise
 4. databases: MS-SQL, Oracle, MySQL
 5. other back-end components: filesystems, directory services ... 
- client-side functionality
 HTML, Hyperlinks, Forms, CSS, JavaScript, VBScript, DOM, Ajax, JSON, Same-Origin Policy, HTML5, Web2.0, Browser extension
- state and sessions

## Encoding Schemes ##
- URL Encoding
contain only 0x20-0x7e
*space % ? & = ; + #*, URL-Encode them when insert them as data into an HTTP request
- Unicode Encoding
- HTML Encoding
- Base64 Encoding
- Hex Encoding
- Remoting and Serialization Frameworks

# 4. Mapping the Applicaiton #
## Enumerating Content and Functionality ##
can be identified via manually, but better advanced techniques.
- Web Spidering
*Burp Suite, WebScarab, Zed Attack Proxy, and CAT*
limitation:
 1. cannot handle unusual navigation mechanisms
 2. links buried cannot be found
 3. fine-grained input validation checks
 4. the same URL may return very different content and functions
 5. run indefinitely
 6. break the authenticated session 
- User-Directed Spidering
navigate the application manually, use tools to record and parse.
**Hack Step:**
>  1. use either Burp or WebScarab as a local proxy
>  2. browse the entire application normally
>  3. review the site map, identify any further content
>  4. run the spider and review the results for any additional content
- Discovering Hidden Content **(PDF116)**

 Brute-Force Techniquesm
>   1. request valid and invalid resources, find how it handles invalid
>   2. automated request for each directory
>   3. capture the response
>   4. iterate above

 Inference from Published Content( content discovery )**(PDF123)**
 Use of Public Information
  1. Search Engine & Web Archives **(PDF126)**
  	 *site:, site:...login, link:, related:*
 
 Leveraging the Web Server *(Wikto)*
- Application Pages Versus Funtional Paths
identify the functionality requested by passing the name of a function in a parameter
- Discovering Hidden Parameters
guess possible parameter, example: *debug\test\hide\source = true\yes\on\1*

## Analyzing the Application ##
- core functionality.
- peripheral behavior. off-site links, error, admin, log, redirects
- core security mechanisms. session state, access control, authentication, logic
- how to process user input. URL, query string parameter, POST data, cookie
- technologies on client side. forms, scripts, cookies, so on
- teconologies on server side. pages, SSL, software, databases, e-mail system

----------

- Identifying Entry Points for User Input
URL File Paths (REST-style), Request parameters (URL query string, message body, HTTP cookies), HTTP Headers(Referer, User-Agent), Out-of-Band Channels

- Identifying Server-Side Technologies
Banner Grabbing (server software version), HTTP Fingerprinting (tool: httprecon), File Extensions, Directory Names, Session Tokens, Third-Party Code Components

- Identifying Server-Side Functionality
Dissect Request, Extrapolate Application Behavior, Isolate Unique Application Behavior

- Mapping the Attack Surface
>  1. Client-side validation — Checks may not be replicated on the server
>  2. Database interaction — SQL injection
>  3. File uploading and downloading — Path traversal vulnerabilities, stored cross-site scripting
>  4. Display of user-supplied data — Cross-site scripting
>  5. Dynamic redirects — Redirection and header injection attacks
>  6. Social networking features — username enumeration, stored cross-site scripting
>  7. Login — Username enumeration, weak passwords, ability to use brute force
>  8. Multistage login — Logic fl aws
>  9. Session state — Predictable tokens, insecure handling of tokens
>  10. Access controls — Horizontal and vertical privilege escalation
>  11. User impersonation functions — Privilege escalation
>  12. Use of cleartext communications — Session hijacking, capture of credentials and other sensitive data
>  13. Off-site links — Leakage of query string parameters in the  Referer header
>  14. Interfaces to external systems — Shortcuts in the handling of sessions and/or access controls
>  15. Error messages — Information leakage
>  16. E-mail interaction — E-mail and/or command injection
>  17. Native code components or interaction — Buffer overfl ows
>  18. Use of third-party application components — Known vulnerabilities
>  19. Identifi able web server software — Common confi guration weaknesses, known software bugs

# 5. Bypassing Client-Side Controls #
## Transmitting Data Via the Client ##
- Hidden Form Fileds
intercept it and edit, then send to server
- HTTP Cookies
- URL parameters
- The Referer Header
- Opaque Data
- The ASP.NET ViewState
MAC protection, Base64

## Capturing User Data: HTML Forms ##
- Length Limits
*If-Modified-Since, If-None-Match*
- Script-Based Validation
disable script in browser, or submit valid input and edit in the proxy (more elegant)
- Disabled Elements
see whether it still has effect

## Capturing User Data: Browser Extensions ##
- Common Browser Extension Technologies
Java applets, Flash, and Silverlight
- Approaches to Browser Extensions
 1. intercept and modify the requests made by the component and the responses received from the server
 2. target the component itself directly and attempt to decompile its bytecode to view the original source, or interact dynamically with the component using a debugger
- Intercepting Traffic from Browser Extensions
 1. Handling Serialized Data
  unpack the serialized content, edit it as required, and reserialize it correctly.
   - Java Serialization **(tool: DSer)**
    *Content-Type: application/x-java-serialized-object*
   - Flash Serialization **(AMF format)**
    *Content-Type: application/x-amf*
   - Silverlight Serialization **(WCF)**
    *Content-Type: application/soap+msbin1* 
 2. Obstacles to Intercepting Traffic from Browser Extensions
   - do not honor your proxy configuration (Chapter 20)
   - do not accept SSL certificate
   - other protocol **(tool: Echo Mirage)**
- Decompiling Browser Extensions
 1. Downloading the Bytecode
   **find the URL for extension's bytecode**
   java: *< applet >*  
   other: *< object > *
    - reveal hidden items like images and sheet files
    - clear cache on browser
 2. Decompiling the Bytecode
   java packaged as *.jar*, silverlight packaged as *.xap*. just rename to *.zip* and unpack them.
   java bytecode in *.class*, silverlight bytecode in *.dll*, flash bytecode in *.swf*, and then decompile as follow:
   - Java Tool **(jad)**
   - Flash Tool **(Flasm, Flare, SWFScan)**
   - silverlight Tool **(.NET Reflector)**
 3. Working on the Source Code
   understand the code
   - Recompile and execute within browser
     java: *javac* flash: *flasm* silverlight: *VS*
     then package them, load into browser **(PDF179)**
   - Recompile and execute outside browser
     execute modified version locally, and replace target value of requests using local outcomes 
   - Manupulating the Original Component Using JavaScript
 4. Coping with Bytecode Obfuscation
    just review public methods, refactoring, obfuscate again (Jode) 
 5. an example **(PDF182)**
- Attaching a Debugger
*JavaSnoop, JSwat, Silverlight Spy*
- Native Client Components
*OllyDbg, IDA Pro*

## Handling Client-Side Data Securely ##
- Transmitting Data Via the Client (unsafe)
can be implemented on the server, or the data should be signed and encrypted.
- Validating Client-Generated Data (unsafe if on client)
- Logging and Alerting
once find illegal data sent to server, log and alert, and can also terminate the session.

# 6. Attacking Authentication #
## Authentication Technologies ##
- HTML forms
- password and physical tokens
- SSL certificate
- smartcard
- HTTP authentication
- NTLM or Kerberos
- Authentication services

## Design Flaws in Authentication Mechanisms ##
- Bad Passwords
- Brute-Forcible Login
- Verbose Failure Messages **(PDF204)**
get valid username
- **Vulnerable Transmission of Credentials**
- **Password Change Functionality**
may have bugs that are restricted at the main login
- **Forgotten Passward Functionality**
password recovery challenge is vulnerable
- **'Remember Me' Functionality**
- User Impersonation Functionality
- Incomplete Validation of Credentials
rarely seen
- Nonunique Usernames
rarely seen
- Predictable Usernames
- Predictable Initial Passwords
- Insecure Distribution of Credentials

## Implementation Flaws in Authentication ##
- Fail-Open Login Mechanisms
submit unexpected data, see difference in response.
- Defects in Multistage Login Mechanisms
 1. may assume that a user who accessed stage 3 must finishes 1 2.
 2. stage 2 may trust data from stage 1
 3. each stage may involve different users

...pdf225

# 9. Attacking Data Stores #
## Injecting into Interpreted Contexts ##

## Injecting into SQL ##
- Exploiting a Basic Vulnerability 
- Injecting into Different Statement Types
*SELECT, WHERE, ORDER BY, INSERT, UPDATE, DELETE*
- Finding SQL Injection Bugs
these methods can tell if SQL Injection Bugs exist.
 1. Injecting into String Data **(PDF334)**