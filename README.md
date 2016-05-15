Magento Important Snippet

Add Js

<layout>  
 <default>
    <reference name="head">
        
        <action method="addJs">
            <script>custom-script.js</script>
        </action>
        
        <action method="addItem">
          	<type>skin_js</type>
          	<name>js/script_name.js</name>
          </action>
       </reference>
    </default>
 </layout>

Add Css

<layout>  
 <default>
    <reference name="head">

	    <action method="addCss">
	    <stylesheet>css/javcustom.css</stylesheet>
	    </action>

	    <action method="addCss”>
	    <type>skin_css</type>
	    <file>css/javcustom.css</file>
	    </action>

	    <action method="addItem">
	    <type>skin_css</type>
	    <name>css/javcustom.css</name>
	    </action>        
        
       </reference>
    </default>
 </layout>

Add Top Links

<?xml version="1.0" encoding="UTF-8"?>
<layout version="0.1.0">
    <default>
        <reference name="root">
        <reference name="top.links">
            <action method="addLink" translate="label title">
                <label>Custom Home Link</label>
                <url>home</url>
                <title>Custom Home Link</title>
                <prepare/>
                <urlParams/>
                <position>10</position>
            </action>
        </reference>
        </reference>
    </default>
</layout>

Add My Account Links

<?xml version="1.0" ?>
<layout version="0.1.0">
    <customer_account translate="label">
        <reference name="customer_account_navigation">
            <action method="addLink"><name>test checkout</name><path>checkout</path><label>Inchoo Checkout</label></action>
        </reference>
    </customer_account>
</layout>

Unset and than update links(top links and my account links)

<reference name="right">                
    <action method="unsetChild"><name>wishlist</name></action>
</reference>



Remove Link from My Account 

For start we’ll have to edit our config.xml file, we have to rewrite Magento’s core file.

 <global>
        <blocks>
            <customer>
                <rewrite>
                    <account_navigation>Inchoo_Core_Block_Customer_Account_Navigation</account_navigation>
                </rewrite>
            </customer>
        </blocks>
    </global>


    class Inchoo_Core_Block_Customer_Account_Navigation extends Mage_Customer_Block_Account_Navigation{

	    public function removeLinkByName($name) {
	    	unset($this->_links[$name]);
	    }
    }

    Last thing we need to do is create layout xml file in app/design/frontend/default/default/layout/inchoo_core.xml:

    <?xml version="1.0" ?>
    <layout version="0.1.0">
	    <customer_account translate="label">
		    <reference name="customer_account_navigation">
			    <action method="removeLinkByName">
			    	<name>billing_agreements</name>
			    </action>
			    <action method="removeLinkByName">
			    	<name>recurring_profiles</name>
			    </action>
			    <action method="removeLinkByName">
			    	<name>tags</name>
			    </action>
			    <action method="removeLinkByName">
			    	<name>wishlist</name>
			    </action>
			    <action method="removeLinkByName">
			    	<name>downloadable_products</name>
			    </action>
		    </reference>
	    </customer_account>
    </layout>


Add custom block and call in appropriate place




Fetching Template Blocks using PHP
<?php
    echo $this->getLayout()->createBlock('catalog/product_list_related')->setTemplate('catalog/product/list/related.phtml')->toHtml();
?>

Fetching CMS Blocks using PHP
<?php
    echo $this->getLayout()->createBlock('cms/block')->setBlockId('contacts_text')->toHtml();
?>


#Sending custom transactional emails in Magento#

Step 1) Adding your template to etc/config.xml to register the email template.

First of all, assuming you already have a basic module setup, add this into app/code/[codePool]/[Namespace]/[Module]/etc/config.xml

<config>
    ...
    <global>
        ...
        <template>
            <email>
                <!-- Give the template a uniqiue name, you'll need to refer to this later when sending the email-->
                <[email_template_name]>
                    <label>[Email Template Label]</label>
                    <file>[email_template_filename].html</file>
                    <type>html</type>
                </[email_template_name]>
            </email>
        </template>
    </global>
</config>

Step 2) Creating your email template!

Awesome, we now have the email template setup, next up we’ll need to create the email template itself. This file needs to be saved in app/locale/[your_locale_or_en_US]/template/email/[email_template_filename].html. The locale code is important, this allows you to create the email in multiple languages, the default locale in Magento is en_US, so saving there is a good place if you’re not sure.

<!--@subject This is the subject line of the email! @-->
<div style="font:11px/1.35em Verdana, Arial, Helvetica, sans-serif;">
  <table cellspacing="0" cellpadding="0" border="0" width="98%" style="margin-top:10px; font:11px/1.35em Verdana, Arial, Helvetica, sans-serif; margin-bottom:10px;">
    <tr>
      <td align="center" valign="top">
        <!-- [ header starts here] -->
          <table cellspacing="0" cellpadding="0" border="0" width="650">
            <tr>
              <td valign="top">
                <p>
                  <a href="{{store url=""}}" style="color:#1E7EC8;"><img src="{{skin url="images/logo_email.gif" _area='frontend'}}" alt="Magento" border="0"/></a>
                </p>
              </td>
            </tr>
          </table>
          <!-- [ middle starts here] -->
          <table cellspacing="0" cellpadding="0" border="0" width="650">
            <tr>
              <td valign="top">
                <p>
                <strong>Dear {{var customer_name}}</strong>,<br/>
                This is the content of your email!
                </p>
              </td>
            </tr>
          </table>
      </td>
    </tr>
  </table>
</div>

Items to note in the above email template:

Your subject looks like a HTML comment:

<!--@subject This is the subject line of the email! @-->
Variables!

Since these templates don’t support PHP directly, we can use this form accessing variables and objects.

Get the current stores URL
{{store url=""}}

Get the skin url, accepts a path, in this case it's the logo image.
{{skin url="images/logo_email.gif"}}

Custom variables, as defined by us. We'll cover this shortly!
{{var customer_name}}
Step 3) Sending the email!

We have our email template configured, and recognised by Magento. Now it’s time to put it all together and send the email!

<?php

// This is the template name from your etc/config.xml
$template_id = '[email_template_name]';

// Who were sending to...
$email_to = 'demo@example.com';
$customer_name   = 'John Doe';

// Load our template by template_id
$email_template  = Mage::getModel('core/email_template')->loadDefault($template_id);

// Here is where we can define custom variables to go in our email template!
$email_template_variables = array(
    'customer_name' => $customerName
    // Other variables for our email template.
);

// I'm using the Store Name as sender name here.
$sender_name = Mage::getStoreConfig(Mage_Core_Model_Store::XML_PATH_STORE_STORE_NAME);
// I'm using the general store contact here as the sender email.
$sender_email = Mage::getStoreConfig('trans_email/ident_general/email');
$email_template->setSenderName($sender_name);
$email_template->setSenderEmail($sender_email);

//Send the email!
$email_template->send($email_to, $customer_name, $email_template_variables);
?>
Conclusion

And that’s it, that is how you send a transactional email in Magento. We can put the code from step 3 in side of a Cron, Observer, or in a custom model to send emails how ever we like. Here are a few use cases:

Automate getting reviews and feedback from customers
Abandon cart emails
Customer assistance (ask if they need any help after browsing the store)
And many more!
