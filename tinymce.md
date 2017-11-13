# TinyMCE

### Paste As Text: 

The forces the CMS to paste as text into the Rich Text Editor, which prevents difficult Microsoft Word characters being inserted. 

Update the following file: `~/config/tinyMceConfig.config`. 

Add the following command in the Commands element:

~~~xml
    <command>
      <umbracoAlias>mcePasteAsText</umbracoAlias>
      <icon>images/editor/paste.gif</icon>
      <tinyMceCommand value="" userInterface="true" frontendCommand="pastetext">pastetext</tinyMceCommand>
      <priority>79</priority>
    </command>
~~~

Add the following configs in the customConfig element. 

~~~xml
 <config key="paste_as_text">true</config>
    <config key="paste_text_sticky">true</config>
    <config key="paste_text_sticky_default">true</config>
~~~