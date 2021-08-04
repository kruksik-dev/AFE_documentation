Communication
+++++++++++++

On this page you will find instructions on how to 
communicate to MCORD HUB via UART and LAN connection.


USB connection
==============

After uploading the prepared version of the micropython, the HUB as a device supports communication with it by default using the serial port. 
To do this, connect to the HUB using a microusb cable. 
Information on how to use a serial connection with the 
``putty`` program can be found `here <https://pbxbook.com/voip/sputty.html>`_.


LAN connection
==============

A JSON server was prepared using the socket package to enable communication with the 
HUB via the lan network. For this purpose, an 
instance of the class responsible for creating the server 
is created in the script:

.. code-block:: python

   serv = Ctlsrv()

TThe default device waits for an external DHCP server to assign an address from the pool. 
In order to check the ip address and network settings use

.. code-block:: python

   serv.get_IP() # (('192.168.10.100','255.255.255.0','192.168.10.255','8.8.8.8'))

To start the server operation, start the thread by calling the *run* method on an instance 
of the server object. As an argument it takes the value of the port on 
which we want our server to work

.. code-block:: python

   serv.run(port)

The server accepts information from the user in the form of JSON, which includes the name of the 
function and its parameters, or a list of parameters

.. code-block:: python

    json_object(['name_of_func',*params])
    json_object(['name_of_func', list])


In order to safely disconnect with our server, send a command containing the word ending connection to it ``!disconnect``

.. code-block:: python

   json_object(['!disconnect'])


Available HUB methods
=====================

The HUB has several predefined functions to control the operation of SiPMs located directly in the boards.


Initialization 
~~~~~~~~~~~~~~~
*The function initializing the operation of the device, setting the initial operating parameters.
The function takes the board's id as a parameter*

.. code-block:: python

   name_of_func = 'init'
   def initialization(obj):
    if isinstance(obj[1], list):
        return loop_for_list_arg(misc.init,obj[1])
    else:
        return ('OK', misc.init(obj[1]))

The function returns the voltage (values on the transducer) that has been set by default on both SIPMs on the board and sets the appropriate offset of this value.

.. code-block:: python

   send_json(['init',9]) # ['OK',[3768,3768]]
   send_json(['init',[9,13]]) # ['OK', {'9': [3768, 3768], '10': [3768, 3768]}]


Turn On SiPM 
~~~~~~~~~~~~
*The function turns on the detector. The function takes the board's id as a parameter*


.. code-block:: python

   name_of_func = 'hvon'
   def turn_on(obj):
    if isinstance(obj[1], list):
        return loop_for_list_arg(misc.HVon,obj[1])
    return('OK', misc.HVon(obj[1]))


The function returns the None object if the command has been correctly executed, otherwise we will get an error message.

.. code-block:: python

   send_json(['hvon',9]) # ['OK', None]
   send_json(['hvon',[9,13]]) # ['OK', {'13': 'ERR', '9': None}]


Setting the operating voltage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*The function allows you to set the operating voltage on SiPMs. As parameters, the function takes the board's id and the voltage values to be set for each of the detectors.*


.. code-block:: python

   name_of_func = 'setdac'
   def setdac(obj):
    if isinstance(obj[1], list):
     return loop_for_afe_list_arg(afedrv.SetDac,obj[1],obj[2],obj[3])
    return ('OK', afedrv.SetDac(obj[1],obj[2],obj[3]))


The function returns the id of the board and the tension that have been set on this board in the form of values on the transducer.


.. code-block:: python

   send_json(['setdac',9,55,55]) # ['OK', [3226, 3226]]
   send_json(['setdac',[9,12],[54,54],[55,55]]) # ['OK', {'12': [3226, 3226], '9': [3497, 3497]}]



Turn Off SiPM
~~~~~~~~~~~~~
*The function turns off the detector. The function takes the board's id as a parameter*


.. code-block:: python

    name_of_func = 'hvoff'
    def turn_off(obj):
     if isinstance(obj[1], list):
        return loop_for_list_arg(misc.HVoff,obj[1])
     return('OK', misc.HVoff(obj[1]))


The function returns the None object if the command has been correctly executed, otherwise we will get an error message.


.. code-block:: python

   send_json(['hvoff',9]) # ['OK', None]
   send_json(['hvoff',[9,13]]) # ['OK', {'9': None, '13': None}]


