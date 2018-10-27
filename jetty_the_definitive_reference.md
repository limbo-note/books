- jetty.home

	The property that defines the location of the jetty distribution, its libs, default modules and default XML files (typically start.jar, lib, etc)

- jetty.base

	The property that defines the location of a specific instance of a jetty server, its configuration, logs and web applications (typically start.ini, start.d, logs and webapps)

# Ⅳ. Jetty Development Guide #
### 21. Embedding
**a HelloWorld Example:**

	import java.io.IOException;

	import javax.servlet.ServletException;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;

	import org.eclipse.jetty.server.Request;
	import org.eclipse.jetty.server.Server;
	import org.eclipse.jetty.server.handler.AbstractHandler;

	public class HelloWorld extends AbstractHandler
	{
	    @Override
	    public void handle( String target,
	                        Request baseRequest,
	                        HttpServletRequest request,
	                        HttpServletResponse response ) throws IOException,
	                                                      ServletException
	    {
	        // Declare response encoding and types
	        response.setContentType("text/html; charset=utf-8");

	        // Declare response status code
	        response.setStatus(HttpServletResponse.SC_OK);

	        // Write back response
	        response.getWriter().println("<h1>Hello World</h1>");

	        // Inform jetty that this request has now been handled
	        baseRequest.setHandled(true);
	    }

	    public static void main( String[] args ) throws Exception
	    {
	        Server server = new Server(8080);
	        server.setHandler(new HelloWorld());

	        server.start();
	        server.join();
	    }
	}

running Jetty in embedded mode means putting an HTTP module into your application, rather than putting your application into an HTTP server.

**steps：**
>1. Create a Server instance.
>2. Add/Configure Connectors.
>3. Add/Configure Handlers and/or Contexts and/or Servlets.
>4. Start the Server.
>5. Wait on the server or do something else with your thread.

**① Creating the Server**  
	
	import org.eclipse.jetty.server.Server;

	/**
	 * The simplest possible Jetty server.
	 */
	public class SimplestServer
	{
	    public static void main( String[] args ) throws Exception
	    {
	        Server server = new Server(8080);
	        server.start();
	        server.dumpStdErr();
	        server.join();
	    }
	}
 runs an HTTP server on port 8080. It is not a very useful server as it has no handlers, and thus returns a 404 error for every request.

**② Using Handlers**  

A handler may:
- Examine/modify the HTTP request.
- Generate the complete HTTP response.
- Call another Handler (see HandlerWrapper).
- Select one or many Handlers to call (see HandlerCollection).

		public class HelloHandler extends AbstractHandler
		{
			@override
			public void handle( String target,
		                        Request baseRequest,
		                        HttpServletRequest request,
		                        HttpServletResponse response ) throws IOException,
		                                                      ServletException
		    {
		        response.setContentType("text/html; charset=utf-8");
		        response.setStatus(HttpServletResponse.SC_OK);

		        PrintWriter out = response.getWriter();

		        out.println("<h1>" + greeting + "</h1>");
		        if (body != null)
		        {
		            out.println(body);
		        }

		        baseRequest.setHandled(true);
		    }
		}

The parameters passed to the handle method are:

- target – the target of the request, which is either a URI or a name from a named dispatcher.
- baseRequest – the Jetty mutable request object, which is always unwrapped.
- request – the immutable request object, which may have been wrapped by a filter or servlet.
- response – the response, which may have been wrapped by a filter or servlet.

The handler sets the response status, content-type, and marks the request as handled before it generates the body of the response using a writer.

add it to a Server instance：

	server.setHandler(new HelloHandler());

many specific handlers: ContextHandler / ServletHandler / RequestLogHandler / StatisticsHandler / ScopedHandler / ResourceHandler 

HandlerContainer: HandlerCollection / HandlerList / HandlerWrapper / ContextHandlerCollection

**③ Embedding Connectors**  
In the previous examples, the Server instance is passed a port number and it internally creates a default instance of a Connector that listens for requests on that port. 

often when embedding Jetty it is desirable to explicitly instantiate and configure one or more Connectors for a Server instance.

	public class OneConnector
	{
	    public static void main( String[] args ) throws Exception
	    {
	        // The Server
	        Server server = new Server();

	        // HTTP connector
	        ServerConnector http = new ServerConnector(server);
	        http.setHost("localhost");
	        http.setPort(8080);
	        http.setIdleTimeout(30000);

	        // Set the connector
	        server.addConnector(http);

	        // Set a handler
	        server.setHandler(new HelloHandler());

	        // Start the server
	        server.start();
	        server.join();
	    }
	}

**④ Embedding Servlets**  

	public class MinimalServlets
	{
	    public static void main( String[] args ) throws Exception
	    {
	        // Create a basic jetty server object that will listen on port 8080.
	        // Note that if you set this to port 0 then a randomly available port
	        // will be assigned that you can either look in the logs for the port,
	        // or programmatically obtain it for use in test cases.
	        Server server = new Server(8080);

	        // The ServletHandler is a dead simple way to create a context handler
	        // that is backed by an instance of a Servlet.
	        // This handler then needs to be registered with the Server object.
	        ServletHandler handler = new ServletHandler();
	        server.setHandler(handler);

	        // Passing in the class for the Servlet allows jetty to instantiate an
	        // instance of that Servlet and mount it on a given context path.

	        // IMPORTANT:
	        // This is a raw Servlet, not a Servlet that has been configured
	        // through a web.xml @WebServlet annotation, or anything similar.
	        handler.addServletWithMapping(HelloServlet.class, "/*");

	        // Start things up!
	        server.start();

	        // The use of server.join() the will make the current thread join and
	        // wait until the server is done executing.
	        // See
	        // http://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html#join()
	        server.join();
	    }

	    @SuppressWarnings("serial")
	    public static class HelloServlet extends HttpServlet
	    {
	        @Override
	        protected void doGet( HttpServletRequest request,
	                              HttpServletResponse response ) throws ServletException,
	                                                            IOException
	        {
	            response.setContentType("text/html");
	            response.setStatus(HttpServletResponse.SC_OK);
	            response.getWriter().println("<h1>Hello from HelloServlet</h1>");
	        }
	    }
	}

**⑤ Embedding Contexts**  
responds only to requests that have a URI prefix that matches the configured context path. 

	public class OneContext
	{
	    public static void main( String[] args ) throws Exception
	    {
	        Server server = new Server( 8080 );

	        // Add a single handler on context "/hello"
	        ContextHandler context = new ContextHandler();
	        context.setContextPath( "/hello" );
	        context.setHandler( new HelloHandler() );

	        // Can be accessed using http://localhost:8080/hello

	        server.setHandler( context );

	        // Start the server
	        server.start();
	        server.join();
	    }
	}

or

	public class ManyContexts
	{
	    public static void main( String[] args ) throws Exception
	    {
	        Server server = new Server(8080);

	        ContextHandler context = new ContextHandler("/");
	        context.setContextPath("/");
	        context.setHandler(new HelloHandler("Root Hello"));

	        ContextHandler contextFR = new ContextHandler("/fr");
	        contextFR.setHandler(new HelloHandler("Bonjoir"));

	        ContextHandler contextIT = new ContextHandler("/it");
	        contextIT.setHandler(new HelloHandler("Bongiorno"));

	        ContextHandler contextV = new ContextHandler("/");
	        contextV.setVirtualHosts(new String[] { "127.0.0.2" });
	        contextV.setHandler(new HelloHandler("Virtual Hello"));

	        ContextHandlerCollection contexts = new ContextHandlerCollection();
	        contexts.setHandlers(new Handler[] { context, contextFR, contextIT,
	                contextV });

	        server.setHandler(contexts);

	        server.start();
	        server.join();
	    }
	}