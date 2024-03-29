Detailed documentation
**********************

Test Setup
----------

Most API methods will only return True or False,
to get a more verbose output and prohibit
making requests to Silverpop
We monkeypatch urllib to print
(url, headers, data) instead of doing any requests.

We create a Fake class to be returned by urlib2.urlopen, 
which will always return a successful silverpop response::

    >>> class Fake(object):
    ...     def read(self): return "<success>true</success>"

In our test method, we print request's url, headers, data (we decode
the urlencoded data for the test) and
return a Fake object::

    >>> import cgi
    >>> def test_urlopen(req):
    ...     print '-'*30 + 'request details' + '-'*30
    ...     print req.get_full_url()
    ...     print req.headers
    ...     xml = dict(cgi.parse_qsl(req.data))['xml']
    ...     print xml
    ...     print '-'*75
    ...     return Fake()
    >>> import urllib2

Finally we patch urllib2.urlopen::

    >>> urllib2.urlopen = test_urlopen

We also define a FakeRequest class to define our request
containing just a form::

    >>> class FakeRequest(dict):
    ...   def __init__(self, **kwargs):
    ...     self.form = kwargs

API Methods
-----------

All api methods can be accessed by importing the module::

 >>> import silverpop


First, we define data needed for the various api_methods.

The URL of the Silverpop Server::

     >>> api_url = 'http://api1.silverpop.com/XMLAPI '

A List Id::

     >>> list_id = 999

An Email Address (we only support email key lists, so this is
our key identifying a newsletter subscriber)::

     >>> email = 'my@email.com'

add_recipient
+++++++++++++


Let's call the add_recipient api with the required attributes::

     >>> silverpop.add_recipient(api_url, list_id, email)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <AddRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <UPDATE_IF_FOUND>true</UPDATE_IF_FOUND>
              <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>my@email.com</VALUE>
              </COLUMN>
            </AddRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

If we provide a list of columns, these will be used in the request, leading to columns in silverpop.

For example, we want to use a custom column **gender**::

    >>> columns = [{'column_name': 'gender', 'column_value': 'male'}, ]

    >>> silverpop.add_recipient(api_url, list_id, email, columns)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <AddRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <UPDATE_IF_FOUND>true</UPDATE_IF_FOUND>
              <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>my@email.com</VALUE>
              </COLUMN>
              <COLUMN>
                <NAME>gender</NAME>
                <VALUE>male</VALUE>
              </COLUMN>
            </AddRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

Note that we have some optional nodes (UPDATE_IF_FOUND, SEND_AUTOREPLY) in the created requests.
These are default. To change this we can use an additional parameter optionals.

Let's call the add_recipient api with the additional optionals attribute to omit the **SEND_AUTOREPLY**::

     >>> custom_optionals = [{'optional_name':'UPDATE_IF_FOUND', 'optional_value':'true'},]
     >>> silverpop.add_recipient(api_url, list_id, email, optionals=custom_optionals)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <AddRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <UPDATE_IF_FOUND>true</UPDATE_IF_FOUND>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>my@email.com</VALUE>
              </COLUMN>
            </AddRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

The **SEND_AUTOREPLY** node is gone.

Of course you can use this together with columns::

    >>> columns = [{'column_name': 'gender', 'column_value': 'male'}, ]

    >>> silverpop.add_recipient(api_url, list_id, email, columns, optionals=custom_optionals)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <AddRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <UPDATE_IF_FOUND>true</UPDATE_IF_FOUND>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>my@email.com</VALUE>
              </COLUMN>
              <COLUMN>
                <NAME>gender</NAME>
                <VALUE>male</VALUE>
              </COLUMN>
            </AddRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

The **SEND_AUTOREPLY** node is gone.

update_recipient
++++++++++++++++

**update_recipient** works similar as **add_recipient**,
but offers to change the recipient's email address, and re-opt-in
an opted out recipient in addition.
for our first test, we are not going to do that::

    >>> old_email = 'my@email.com'

Let's call the update_recipient api with the required attributes::

     >>> silverpop.update_recipient(api_url, list_id, old_email)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <UpdateRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <OLD_EMAIL>my@email.com</OLD_EMAIL>
              <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
            </UpdateRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

If we provide a list of columns, these will be used in the request, leading to columns in silverpop.
This is also used to change the recipient's email address.

For example, we want to change the **email address**, therefore we need to specify one column **EMAIL**::

    >>> columns = [{'column_name': 'EMAIL', 'column_value': 'new@email.com'}, ]

    >>> silverpop.update_recipient(api_url, list_id, old_email, columns)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <UpdateRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <OLD_EMAIL>my@email.com</OLD_EMAIL>
              <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>new@email.com</VALUE>
              </COLUMN>
            </UpdateRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

Another common use case is to to re-opt-in an opted-out recipient.

Therefore we provide a column **OPT_OUT**, which when set to **False**
will re-opt-in the recipient::


    >>> columns = [{'column_name': 'OPT_OUT', 'column_value': 'False'}, ]

    >>> silverpop.update_recipient(api_url, list_id, old_email, columns)
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <UpdateRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <OLD_EMAIL>my@email.com</OLD_EMAIL>
              <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
              <COLUMN>
                <NAME>OPT_OUT</NAME>
                <VALUE>False</VALUE>
              </COLUMN>
            </UpdateRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

Note that we have an optional node ( SEND_AUTOREPLY ) in the created requests.
This is default. To change this we can use an additional parameter optionals.


    >>> old_email = 'my@email.com'

Let's call the update_recipient api with the additional optionals attribute to omit the **SEND_AUTOREPLY**::

     >>> silverpop.update_recipient(api_url, list_id, old_email, optionals=[])
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <UpdateRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <OLD_EMAIL>my@email.com</OLD_EMAIL>
            </UpdateRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

The **SEND_AUTOREPLY** node is gone.

Of course you can use this together with columns::

    >>> columns = [{'column_name': 'EMAIL', 'column_value': 'new@email.com'}, ]

    >>> silverpop.update_recipient(api_url, list_id, old_email, columns, optionals=[])
        ------------------------------request details------------------------------
        http://api1.silverpop.com/XMLAPI
        {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
        <Envelope>
          <Body>
            <UpdateRecipient>
              <LIST_ID>999</LIST_ID>
              <CREATED_FROM>2</CREATED_FROM>
              <OLD_EMAIL>my@email.com</OLD_EMAIL>
              <COLUMN>
                <NAME>EMAIL</NAME>
                <VALUE>new@email.com</VALUE>
              </COLUMN>
            </UpdateRecipient>
          </Body>
        </Envelope>
        ---------------------------------------------------------------------------
        True

The **SEND_AUTOREPLY** node is gone.

opt_in_recipient
++++++++++++++++

This method is a wrapper around update_recipient_data,
for explicitly opting in a recipient.

Let's call the opt_in_recipient api with the required attributes::

    >>> silverpop.opt_in_recipient(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <UpdateRecipient>
          <LIST_ID>999</LIST_ID>
          <CREATED_FROM>2</CREATED_FROM>
          <OLD_EMAIL>my@email.com</OLD_EMAIL>
          <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
          <COLUMN>
            <NAME>OPT_OUT</NAME>
            <VALUE>False</VALUE>
          </COLUMN>
        </UpdateRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

If we provide a list of columns, these will be used in the request, 
leading to columns in silverpop.
This can be also used to change the recipient's email address.
For example, we want to change the **email address**.
Therefore we need to specify one column **EMAIL**::

    >>> columns = [{'column_name': 'EMAIL', 'column_value': 'new@email.com'}, ]

    >>> silverpop.opt_in_recipient(api_url, list_id, email, columns)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <UpdateRecipient>
          <LIST_ID>999</LIST_ID>
          <CREATED_FROM>2</CREATED_FROM>
          <OLD_EMAIL>my@email.com</OLD_EMAIL>
          <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
          <COLUMN>
            <NAME>OPT_OUT</NAME>
            <VALUE>False</VALUE>
          </COLUMN>
          <COLUMN>
            <NAME>EMAIL</NAME>
            <VALUE>new@email.com</VALUE>
          </COLUMN>
        </UpdateRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

If the user specifies an **OPT_OUT** column, this will be ignored,
as we always want to opt in.

Let's define the to be ignored column::

    >>> columns = [{'column_name': 'OPT_OUT', 'column_value': 'some value'}, ]

Notice how the value of **OPT_OUT** is still **False**::

    >>> silverpop.opt_in_recipient(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <UpdateRecipient>
          <LIST_ID>999</LIST_ID>
          <CREATED_FROM>2</CREATED_FROM>
          <OLD_EMAIL>my@email.com</OLD_EMAIL>
          <SEND_AUTOREPLY>true</SEND_AUTOREPLY>
          <COLUMN>
            <NAME>OPT_OUT</NAME>
            <VALUE>False</VALUE>
          </COLUMN>
        </UpdateRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

Note that we have an optional node ( SEND_AUTOREPLY ) in the created requests.
This is default. To change this we can use an additional parameter optionals.


    >>> old_email = 'my@email.com'

Let's call the opt_in_recipient api with the additional optionals attribute to omit the **SEND_AUTOREPLY**::

    >>> silverpop.opt_in_recipient(api_url, list_id, email, optionals=[])
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <UpdateRecipient>
          <LIST_ID>999</LIST_ID>
          <CREATED_FROM>2</CREATED_FROM>
          <OLD_EMAIL>my@email.com</OLD_EMAIL>
          <COLUMN>
            <NAME>OPT_OUT</NAME>
            <VALUE>False</VALUE>
          </COLUMN>
        </UpdateRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

The **SEND_AUTOREPLY** node is gone.

Of course you can use this together with columns::

    >>> columns = [{'column_name': 'EMAIL', 'column_value': 'new@email.com'}, ]

    >>> silverpop.opt_in_recipient(api_url, list_id, email, columns, optionals=[])
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <UpdateRecipient>
          <LIST_ID>999</LIST_ID>
          <CREATED_FROM>2</CREATED_FROM>
          <OLD_EMAIL>my@email.com</OLD_EMAIL>
          <COLUMN>
            <NAME>OPT_OUT</NAME>
            <VALUE>False</VALUE>
          </COLUMN>
          <COLUMN>
            <NAME>EMAIL</NAME>
            <VALUE>new@email.com</VALUE>
          </COLUMN>
        </UpdateRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

The **SEND_AUTOREPLY** node is gone.

opt_out_recipient
+++++++++++++++++

Let's call the opt_out_recipient api with the required attributes::

    >>> silverpop.opt_out_recipient(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <OptOutRecipient>
          <LIST_ID>999</LIST_ID>
          <EMAIL>my@email.com</EMAIL>
        </OptOutRecipient>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

is_opted_in
+++++++++++

This method returns **True** when the specified email is a
non-opted out recipient of a list. 
If the list doesn't exist, or
the user isn't a recipient of an existing list, or 
the user is opted out form the specified list, 
it should return **False**.

For this test we need to interpret the various responses,
which we could get from silverpop, so we are going to change 
in more detail, so we change our Fake class simulating, the different Silverpop
responses.

First, we assume that we have specified a non existing list_id
(is_opted_in should return False)::

    >>> class Fake(object):
    ...     def read(self): return """
    ...         <Envelope>
    ...           <Body>
    ...             <RESULT>
    ...                 <SUCCESS>false</SUCCESS>
    ...             </RESULT>
    ...             <Fault>
    ...               <Request/>
    ...               <FaultCode/>
    ...               <FaultString>
    ...                 <![CDATA[List with id 999 Does Not Exist.]]>
    ...               </FaultString>
    ...               <detail>
    ...                 <error>
    ...                   <errorid>108</errorid>
    ...                   <module/>
    ...                   <class>SP.ListManager</class>
    ...                   <method/>
    ...                 </error>
    ...                </detail>
    ...               </Fault>
    ...             </Body>
    ...         </Envelope>
    ...         """

    >>> silverpop.is_opted_in(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    False

Now, we specify a list_id which is not valid (it must be an integer)::

    >>> class Fake(object):
    ...     def read(self): return """
    ...         <Envelope>
    ...           <Body>
    ...             <RESULT>
    ...                 <SUCCESS>false</SUCCESS>
    ...             </RESULT>
    ...             <Fault>
    ...               <Request/>
    ...               <FaultCode/>
    ...               <FaultString>
    ...                 <![CDATA[List ID is not valid.]]>
    ...               </FaultString>
    ...               <detail>
    ...                 <error>
    ...                   <errorid>106</errorid>
    ...                   <module/>
    ...                   <class>SP.ListManager</class>
    ...                   <method/>
    ...                 </error>
    ...                </detail>
    ...               </Fault>
    ...             </Body>
    ...         </Envelope>
    ...         """

    >>> silverpop.is_opted_in(api_url, 'NOT AN INTEGER', email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>NOT AN INTEGER</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    False

Now, we assume the provided email is not a member of the list::

    >>> class Fake(object):
    ...     def read(self): return """
    ...         <Envelope>
    ...           <Body>
    ...             <RESULT>
    ...                 <SUCCESS>false</SUCCESS>
    ...             </RESULT>
    ...             <Fault>
    ...               <Request/>
    ...               <FaultCode/>
    ...               <FaultString>
    ...                 <![CDATA[Recipient is not a member of the list.]]>
    ...               </FaultString>
    ...               <detail>
    ...                 <error>
    ...                   <errorid>128</errorid>
    ...                   <module/>
    ...                   <class>SP.ListManager</class>
    ...                   <method/>
    ...                 </error>
    ...                </detail>
    ...               </Fault>
    ...             </Body>
    ...         </Envelope>
    ...         """

    >>> silverpop.is_opted_in(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    False

Now, we assume the provided email is a member of the list, but the member is opted out
(leading to a value in the **<OptedOut>** tag)::

    >>> class Fake(object):
    ...     def read(self): return """
    ...         <Envelope>
    ...           <Body>
    ...             <RESULT>
    ...               <SUCCESS>TRUE</SUCCESS>
    ...               <EMAIL>my@email.com</EMAIL>
    ...               <Email>my@email.com</Email>
    ...               <RecipientId>5</RecipientId>
    ...               <EmailType>0</EmailType>
    ...               <LastModified>5/11/09 9:43 AM</LastModified>
    ...               <CreatedFrom>2</CreatedFrom>
    ...               <OptedIn>3/26/09 10:29 AM</OptedIn>
    ...               <OptedOut>5/11/09 9:43 AM</OptedOut>
    ...               <COLUMNS>
    ...                 <COLUMN>
    ...                   <NAME>State</NAME>
    ...                   <VALUE>Germany</VALUE>
    ...                 </COLUMN>
    ...               </COLUMNS>
    ...             </RESULT>
    ...           </Body>
    ...         </Envelope>
    ...         """

    >>> silverpop.is_opted_in(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    False


Finally, we assume the provided email is a member of the list, and the member isn't opted out
(leading to an empty **<OptedOut/>** tag)::

    >>> class Fake(object):
    ...     def read(self): return """
    ...         <Envelope>
    ...           <Body>
    ...             <RESULT>
    ...               <SUCCESS>TRUE</SUCCESS>
    ...               <EMAIL>my@email.com</EMAIL>
    ...               <Email>my@email.com</Email>
    ...               <RecipientId>5</RecipientId>
    ...               <EmailType>0</EmailType>
    ...               <LastModified>5/11/09 9:43 AM</LastModified>
    ...               <CreatedFrom>2</CreatedFrom>
    ...               <OptedIn>3/26/09 10:29 AM</OptedIn>
    ...               <OptedOut/>
    ...               <COLUMNS>
    ...                 <COLUMN>
    ...                   <NAME>State</NAME>
    ...                   <VALUE>Germany</VALUE>
    ...                 </COLUMN>
    ...               </COLUMNS>
    ...             </RESULT>
    ...           </Body>
    ...         </Envelope>
    ...         """

    >>> silverpop.is_opted_in(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    True

select_recipient_data
+++++++++++++++++++++

For this test, we want to have a simple xml as reply from Silverpop, 
as we don't process the response further::

    >>> class Fake(object):
    ...     def read(self): return "<silverpop_response>true</silverpop_response>"


    >>> silverpop.select_recipient_data(api_url, list_id, email)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    '<silverpop_response>true</silverpop_response>'

If we provide a column, this will be used in the request
(use for non email key lists only).


For example, we have a custom key **USER_ID**::

    >>> column = {'column_name': 'USER_ID', 'column_value': '4711'}

    >>> silverpop.select_recipient_data(api_url, list_id, email, column)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <SelectRecipientData>
           <LIST_ID>999</LIST_ID>
           <EMAIL>my@email.com</EMAIL>
             <COLUMN>
               <NAME>USER_ID</NAME>
               <VALUE>4711</VALUE>
             </COLUMN>
        </SelectRecipientData>
      </Body>
    </Envelope>
    ---------------------------------------------------------------------------
    '<silverpop_response>true</silverpop_response>'

xml_request
+++++++++++

The Silverpop XML API offers a quite large variaty of commands,
of which we only implement a subset.
If you need to make different requests you can use this method to
submit custom xml to Silverpop. As result, you will get the Silverpop
Response, which is also xml.

Imagine, we want to use the **ForwardToFrient** xml command. 

Let's define the custom xml::

    >>> xml = """<Envelope>
    ...   <Body>
    ...     <ForwardToFriend>
    ...       <SENDER_EMAIL>bob@bob.com</SENDER_EMAIL>
    ...       <r>5</r>
    ...       <m>10</m>
    ...       <RECIPIENTS>jane@jane.com</RECIPIENTS>
    ...       <MESSAGE>Forwarded: Check this out, I just got that</MESSAGE>
    ...     </ForwardToFriend>
    ...   </Body>
    ...  </Envelope>"""

The xml is sent to Silverpop and we get the response back, for further processing::

    >>> silverpop.xml_request(api_url, xml)
    ------------------------------request details------------------------------
    http://api1.silverpop.com/XMLAPI
    {'Content-type': 'application/x-www-form-urlencoded;charset=UTF-8'}
    <Envelope>
      <Body>
        <ForwardToFriend>
          <SENDER_EMAIL>bob@bob.com</SENDER_EMAIL>
          <r>5</r>
          <m>10</m>
          <RECIPIENTS>jane@jane.com</RECIPIENTS>
          <MESSAGE>Forwarded: Check this out, I just got that</MESSAGE>
        </ForwardToFriend>
      </Body>
     </Envelope>
    ---------------------------------------------------------------------------
    '<silverpop_response>true</silverpop_response>'
