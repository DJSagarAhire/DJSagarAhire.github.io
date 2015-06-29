.. title: A "Hello World" Webapp in Flask
.. slug: a-hello-world-webapp-in-flask
.. date: 2015-06-29 22:40:00 UTC+05:30
.. tags: mathjax, python, flask, web
.. link: 
.. description: 
.. type: text

In this post, we have a look at the Flask web framework for creating web applications in Python, and demonstrate its use by creating an example "Hello World" webapp which shows the basic components of a web application in Flask.

.. TEASER_END

Web Applications (or "Webapps")
-------------------------------

So what is a web application? Put simply, a web application is an application that runs in a web browser. Of course, things are not really that simple! There are several ways to view webapps and tons of books and web pages are written on this subject. I'll summarize the concept of a webapp by breaking it down into its components.

Usually, a web application has two parts: the Client side and the Server side. The *Client side* of the webapp is the part which is actually sent over the Internet by the server to a web browser when a user requests the page. On the other hand, the *Server Side* of the webapp is the part which resides on the server, waiting for the client to perform an action and take actions of its own when it receives a client request.

The code that executes at the client essentially takes the form of a web page. Webpages are made by combining the following three technologies, each of which performs a separate function:

1. **HTML**: HyperText Markup Language (HTML) is a markup language which specifies the content of the page. While I assume everybody who is reading this has at least a basic knowledge of HTML, a common misconception needs to be cleared at this point - HTML does *not* indicate the presentation of the content; it only states the content itself. The abundance of styling tags in the now-deprecated HTML 4 created this problem, which has largely been fixed in HTML 5. Which brings us to...

2. **CSS**: Cascading Style Sheets (CSS) is a language that enables specification of styling for the content in a web page. Style can be applied in any one of three possible locations: Within the ``style`` attribute of an HTML tag, applicable only to the tag; Within the ``<head>`` tag of a document, applicable to the entire document; or in an external CSS document, available to all documents the CSS file is imported in.

3. **Javascript**: While HTML and CSS can create and present content respectively, once the Webpage has been sent from the server, there is no way to modify it! Javascript solves this problem and enables modification of webpages by running code which is executed *at the client*. This allows web pages to be made dynamic, and due to the explosion of webapps, has become one of the most widely used programming languages today.

On the other hand, the code that executes at the server waits for a request, and once a request arrives from the client, performs actions based on it. This code runs on the server, and is often referred to as the "backend" (as opposed to the client side, which is called the "frontend"). It is possible to write code for this in essentially any programming language, although frameworks have been made in several languages that make it easier to write backend code. Languages popular for backend development include Java (with frameworks like `Spring <http://projects.spring.io/spring-framework/>`_ and `Struts <https://struts.apache.org/>`_), Ruby (with the highly popular `Rails <http://rubyonrails.org/>`_ framework), PHP (with `vanilla PHP <http://php.net/>`_ as well as frameworks like `Laravel <http://laravel.com/>`_), Javascript (with `Node.js <https://nodejs.org/>`_ for server-side code) and, of course, Python (with `Django <https://www.djangoproject.com/>`_ and Flask among others).

Flask: An Introduction
----------------------

`Flask <http://flask.pocoo.org/>`_ is a Python web framework based on Werkzeug, Jinja 2 and "good intentions". It is one of the two leading web frameworks for Python (the other being Django). It is interesting to contrast the approaches taken by the two frameworks. While Django supplies all the peripheral components required for web development such as its own ORM (for communicating with a database), Flask chooses to not impose itself, giving the bare essentials while asking the programmer to make a choice for the rest of the components. While this makes Django a great choice if it suits your workflow, Flask is great if you require flexibility. It is also a really good choice for learning web development as you get to know what is happening under the hood.

With that out of the way, we can start developing out first web application using Flask!

Step 1: Hello Flask
-------------------

The very first version of our web page will simply create a web page with the words "Hello World" in it and send it back to the client. The code for that is as follows:

File: ``app.py``

.. code:: python
    :number-lines:

    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def index():
        return "Hello World!"

    if __name__ == "__main__":
        app.run()

You can punch in this code in your favourite text editor and save it as a file with a ``.py`` extension (You are advised to create a separate directory, as web applications are known to balloon in size and you'd want to keep all its files together). Then open a command prompt and type in:

.. code::

    python app.py

Flask will start up a local server and specify its address (usually ``http://127.0.0.1:5000/``). Go there and you'll see our webpage shouting out "Hello World!" in all its glory!

So, what just happened? Let's walk through the code we just wrote. Line 1 imports the ``Flask`` application constructor from the ``flask`` module. Then, using it, we create an object representing our application in Line 3. In Line 5, we define a *route* for the web application, in this case, ``/``. Thus, if the URL of the application is ``www.example.com``, then going to ``www.example.com/`` will invoke the function defined for the route. If the route were ``/index``, then its corresponding URL would have been ``www.example.com/index``. The next two lines state what happens when a user goes to the specified route - he is returned the string "Hello World". Finally, ``app.run()`` runs the Flask server when the script is invoked from the command line, the way we just did.

Well, that wasn't so hard, was it? But that wasn't very impressive either. We want to be able to write more than just "Hello World"! Also, where's the HTML and/or CSS? For that, we move on to the next version of our "Hello World" webapp.

Step 2: Hello Templates
-----------------------

As we saw in the previous section, once the user visits a route in our app, the function associated with it is invoked and whatever is specified in the ``return`` statement is sent back to the requesting browser. So, if we need to send back a pretty HTML page, we need to return HTML from the function.

One way to do this is to write HTML code within the Python file itself, treating it as a string. While this is indeed possible, this would be an extremely ugly and inflexible approach because the webpage would be tied to the Python code. And what if you need to return the same HTML with a very few minor changes from another route? You may need to copy over the entire HTML code within your Python script, and in general this would create a maintainability nightmare. A solution to this problem is Templates.

*Templates* are files where you can write HTML code and simply ask Flask to render them whenever required. This way, the HTML is separated from the Python program and can be used in multiple places in the code without creating problems. All you need to do in the Python program is call the ``render_template()`` function, passing in the HTML template file, and Flask will render the template for you as a Python string, ready to be sent over to the client.

Templates need to be created in a special directory ``\templates`` inside the project. So, what are we waiting for? Let's create one!

File: ``templates\index.html``

.. code:: html

    <!DOCTYPE html>

    <html>
        <head>
	    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css">
	    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"></script>
	    <title>Hello World!</title>
	</head>

	<body>
	    <h1>Hello World!</h1>
	</body>
    </html>

This is simply a standard HTML page. The ``<head>`` tag imports the `Bootstrap <http://getbootstrap.com/>`_ web design framework via a CDN. Even though we aren't explicitly using any of its features here, its default settings give the webpage a nice, pleasing look.

To bring this template into action in our Flask webapp, we need to modify our Python file as follows:

File: ``app.py``

.. code:: python
    :number-lines:

    from flask import Flask, render_template

    app = Flask(__name__)

    @app.route('/')
    def index():
        return render_template("index.html")

    if __name__ == "__main__":
        app.run()

The only modifications required are importing of the function ``render_template`` from the ``flask`` module, and then calling it in the ``index()`` function rather than returning just a string.

On running this version of the Python script, instead of just a string, you get the HTML page, complete with header formatting!

However, this isn't very useful either - it's still a static web page. We still cannot react to user input or make decisions based on certain factors. For that, we move on to the next version of our Hello World app!

Step 3: Hello World!
--------------------

So far our application has no ability to respond to the requests of the outside world as it is still an unchanging webpage. However, in this version of the app, we introduce the real power of Flask, which allows an application to accept user input and respond to it.

One way of accepting user input in a webpage is a form. So let's add a form to our template page which asks the user his/her name. Add the following form in our template file before the closing ``</body>`` tag:

File: ``templates\index.html``

.. code:: html

    <form class="form-inline" action="/">
        <label for="txtName">Enter your name:</label>
	<input type="text" name="name" id="txtName" />

	<button type="submit" class="btn btn-default">Submit</button>
    </form>

While this form won't do anything just yet because we haven't handled the request yet, you can notice that when we click on Submit, we are returned back to the same page (due to the form's ``action`` attribute). This is set up so that we can now handle the request by modifying this very page!

Note that by default, whenever a form gets submitted, it sends a request of type ``GET`` to the server, packing in the values entered in the form using the ``name`` attribute of the form inputs. Flask provides a way to access these *arguments* of the GET request via the ``request`` object. While the ``request`` object is vast and extremely powerful, the only information we need right now from it is the arguments of the GET request, which can be obtained by using the ``request.args.get()`` function and passing in the name of the form control. If corresponding data does not exist, the function returns ``None``.

This data from the request can then be passed onward to the template using keyword arguments on the ``render_template()`` function, so that the template can then display it! Let's make the appropriate changes to our Python file to make that happen:

File: ``app.py``

.. code:: python
    :number-lines:

    from flask import Flask, render_template, request

    app = Flask(__name__)

    @app.route('/')
    def index():
        return render_template("index.html", name=request.args.get("name"))

    if __name__ == "__main__":
        app.run()

Once again, the changes are extremely minimal - one import and one modification to the ``render_template()`` function call. The call to ``request.args.get("name")`` looks for data in the GET request with the name ``name`` and if such data is found, it is returned, else it returns ``None``. This is then passed as an argument to ``render_template()`` with the name ``name`` (any name is fine).

Now, the template has received this data. How do we incorporate this user-submitted data inside our HTML page? This is where we find the power behind Flask templates.

Once the template receives data using a keyword argument to ``render_template()``, this data becomes available to it, and can be printed within the template using special syntax! For example, in order to print the contents of the ``name`` keyword argument, you can use ``{{ name }}``. This can be done anywhere in the template file and ``render_template()`` will do the job of replacing the brace-bracketed statement with the contents of the keyword argument passed to the function.

In addition, it is also possible to use conditions such as ``if`` and even loops such as ``for`` within templates! These can be used to modify the contents of the webpage depending on requirements. For instance, if an error occurs, you can create a conditional block with ``{% if error %}`` and include an error message.

We now modify the template to display the user's name when a name is entered, or simply "Hello World" otherwise:

File: ``templates\index.html``

.. code:: html

    <!DOCTYPE html>

    <html>
        <head>
	    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css">
	    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"></script>
	    <title>Hello World!</title>
	</head>

	<body>
	    {% if name %}
	    <h1>Hello {{ name }}!</h1>
	    {% else %}
	    <h1>Hello World!</h1>
	    {% endif %}

	    <form class="form-inline" action="/">
	        <label for="txtName">Enter your name:</label>
		<input type="text" name="name" id="txtName" />

		<button type="submit" class="btn btn-default">Submit</button>
	    </form>
	</body>
    </html>

The ``{% if name %}`` statement handles the case when the name is not entered, and thus, the ``name`` variable is ``None``. This happens when the page is first loaded, for example.

Now, when the Python script is run, you can enter your name, and see the server respond by greeting you!

**Code**

The code is available on the `github repository <https://github.com/DJSagarAhire/blog-code/tree/master/004>`_ accompanying the site.

**Exercise**

Modify the Hello World webapp to greet the user with "Good Morning", "Good Afternoon", "Good Evening" or "Good Night", depending on the time at the server.

