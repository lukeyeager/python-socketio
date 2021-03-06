.. socketio documentation master file, created by
   sphinx-quickstart on Sat Jun 13 23:41:23 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

socketio documentation
======================

:ref:`genindex` | :ref:`modindex` | :ref:`search`

This project implements an Socket.IO server that can run standalone or
integrated with a Python WSGI application. The following are some of its
features:

- Fully compatible with the Javascript
  `socket.io-client <https://github.com/Automattic/socket.io-client>`_ library.
- Compatible with Python 2.7 and Python 3.3+.
- Based on `Eventlet <http://eventlet.net/>`_, enabling large number of
  clients even on modest hardware.
- Includes a WSGI middleware that integrates Socket.IO traffic with standard
  WSGI applications.
- Broadcasting of messages to all or a subset of the connected clients.
- Event-based architecture implemented with decorators that hides the
  details of the protocol.
- Support for HTTP long-polling and WebSocket transports.
- Support for XHR2 and XHR browsers.
- Support for text and binary messages.
- Support for gzip and deflate HTTP compression.
- Configurable CORS responses to avoid cross-origin problems with browsers.

What is Socket.IO?
------------------

Socket.IO is a transport protocol that enables real-time bidirectional
event-based communication between clients (typically web browsers) and a
server. The official implementations of the client and server components are
written in JavaScript.

Getting Started
---------------

The following is a basic example of a Socket.IO server that uses Flask to
deploy the client code to the browser::

    import socketio
    import eventlet
    from flask import Flask, render_template

    sio = socketio.Server()
    app = Flask(__name__)

    @app.route('/')
    def index():
        """Serve the client-side application."""
        return render_template('index.html')

    @sio.on('connect')
    def connect(sid, environ):
        print('connect ', sid)

    @sio.on('my message')
    def message(sid, data):
        print('message ', data)

    @sio.on('disconnect')
    def disconnect(sid):
        print('disconnect ', sid)

    if __name__ == '__main__':
        # wrap Flask application with socketio's middleware
        app = socketio.Middleware(sio, app)

        # deploy as an eventlet WSGI server
        eventlet.wsgi.server(eventlet.listen(('', 8000)), app)

The client-side application must include the
`socket.io-client <https://github.com/Automattic/socket.io-client>`_ library
(versions 1.3.5 or newer recommended).

Each time a client connects to the server the ``connect`` event handler is
invoked with the ``sid`` (session ID) assigned to the connection and the WSGI
environment dictionary. The server can inspect authentication or other headers
to decide if the client is allowed to connect. To reject a client the handler
must return ``False``.

When the client sends an event to the server the appropriate event handler is
invoked with the ``sid`` and the message. The application can define as many
events as needed and associate them with event handlers. An event is defined
simply by a name.

When a connection with a client is broken, the ``disconnect`` event is called,
allowing the application to perform cleanup.

Rooms
-----

Because Socket.IO is a bidirectional protocol, the server can send messages to
any connected client at any time. To make it easy to address specific subsets
of clients, the application can put clients into rooms, and then address
messages to all the clients in a room.

When clients first connect, they are assigned to their own rooms, named with
the session ID (the ``sid`` argument passed to all event handlers). The
application is free to create additional rooms and manage which clients are in
them using the :func:`socketio.Server.enter_room` and
:func:`socketio.Server.leave_room` methods. Clients can be in as many rooms as
needed and can be moved between rooms as often as necessary. The individual
rooms assigned to clients when they connect are not special in any way, the
application is free to add or remove clients from them, though once it does
that it will lose the ability to address individual clients.

::

    @sio.on('enter room')
    def enter_room(sid, data):
        sio.enter_room(sid, data['room'])

    @sio.on('enter room')
    def leave_room(sid, data):
        sio.leave_room(sid, data['room'])

The :func:`socketio.Server.emit` method takes an event name, a message payload
of type ``str``, ``bytes``, ``list`` or ``dict``, and the recipient room. To
address an individual client, the ``sid`` of that client should be given as
room (assuming the application did not alter these initial rooms). To address
all connected clients, the ``room`` argument should be omitted.

::

    @sio.on('my message')
    def message(sid, data):
        print('message ', data)
        sio.emit('my reply', data, room='my room')

Often when broadcasting a message to group of users in a room, it is desirable
that the sender does not receive its own message. The
:func:`socketio.Server.emit` method provides an optional ``skip_sid`` argument
to specify a client that should be skipped during the broadcast.

::

    @sio.on('my message')
    def message(sid, data):
        print('message ', data)
        sio.emit('my reply', data, room='my room', skip_sid=sid)

Responses
---------

When a client sends an event to the server, it can optionally provide a
callback function, to be invoked with a response provided by the server. The
server can provide a response simply by returning it from the corresponding
event handler.

::

    @sio.on('my event', namespace='/chat')
    def my_event_handler(sid, data):
        # handle the message
        return "OK", 123

The event handler can return a single value, or a tuple with several values.
The callback function on the client side will be invoked with these returned
values as arguments.

Callbacks
---------

The server can also request a response to an event sent to a client. The
:func:`socketio.Server.emit` method has an optional ``callback`` argument that
can be set to a callable. When this argument is given, the callable will be
invoked with the arguments returned by the client as a response.

Using callback functions when broadcasting to multiple clients is not
recommended, as the callback function will be invoked once for each client
that received the message.

Namespaces
----------

The Socket.IO protocol supports multiple logical connections, all multiplexed
on the same physical connection. Clients can open multiple connections by
specifying a different *namespace* on each. A namespace is given by the client
as a pathname following the hostname and port. For example, connecting to
*http://example.com:8000/chat* would open a connection to the namespace
*/chat*.

Event handlers and rooms are maintained separately for each namespace, so it is
important that applications that use multiple namespaces specify the correct
namespace when setting up event handlers and rooms, using the ``namespace``
argument available in All the methods in the :class:`socketio.Server` class.

When the ``namespace`` argument is omitted, set to ``None`` or to ``'/'``, the
default namespace is used.

API Reference
-------------

.. toctree::
   :maxdepth: 2

.. module:: socketio

.. autoclass:: Middleware
   :members:

.. autoclass:: Server
   :members:
