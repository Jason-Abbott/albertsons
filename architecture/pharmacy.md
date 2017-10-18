# Savon Pharmacy
Front end documentation
prepared by *Jason Abbott*,
October 4, 2000


## Scope and Definitions
The purpose of this document is to describe the architecture of the front end.  By “front end” we mean the VBScript contained in .asp and .inc files, and Visual Basic (VB) DLLs.  Tables and stored procedures are not discussed here.  By “architecture” we mean that the documentation does not describe every line of code but rather paints on overall picture of program execution and standards that should allow problems to be isolated to the page or module level.  Further isolation will depend upon the comments present within the code itself.

## Background
Front end development was originally outsourced to Blir, L.L.C., with development primarily by Michael Dwyer and Shane Martin.    Much of the architecture delivered by Blir was reworked for performance, consistency and efficiency reasons by Jason Abbott and Zeb Olsen of Albertsons, such that little remains of the original design and methodology.  This documentation, therefore, supercedes the documentation provided by Blir.  Specific changes will be detailed throughout this document.

## Languages used
The existing Albertsons commerce sites typically use JavaScript only for specialized form submissions (e.g. adding products to the shopping cart) and VBScript only to call on VB methods, wherein the majority of the logic, including generation of HTML and form validation, resides.  The Savon pharmacy differs from this model.  As delivered by Blir, a substantial percentage of the program execution occurs within VBScript.  VB was used primarily for form validation and for passing values to and retrieving recordsets from stored procedures.  JavaScript was used very little.

Because of complications that arose early on with Blir’s VB validation, we decided to use JavaScript for client-side validation.  This approach was substantially faster (no page repost required) and more flexible.  Removing this function from the VB meant that the VB was little more than a data conduit.  The user interface layer was passed a stored procedure name with parameters; it passed those to the business layer which in turn passed them to the data layer, all with no modifications.  Finally it passed the same information to the Albertsons common data layer.  To avoid unnecessary overhead, we decided, where possible, to pass the stored procedure and arguments directly to the Albertsons data layer, bypassing the Blir VB since, with validation removed, it served no purpose.

This shortcut was only possible for data retrieval.  Because of Blir’s special method of reading form values to populate the list of parameters passed with the stored procedure, data inserts and updates still use the Blir VB, as will be detailed below.

## Form Validation
The majority of form validation is performed on-the-fly (client-side) with JavaScript.  This is a significant departure from Blir’s design.  Each form needs two parts for validation: 1) an object containing the field names and the type of validation to perform, and 2) a reference to the file containing the validation functions.  Consider the following example:

```
var oFields = {
	frmSalutation:{desc:"Title",type:"Select",req:1},
	frmFirstName:{desc:"First Name",type:"String",req:1},
	frmLastName:{desc:"Last Name",type:"String",req:1},
	frmBirthDate:{desc:"Birth Date",type:"Date",req:1},
	frmCompanyName:{desc:"Company Name",type:"String",req:0}
};
```

The object name must always be oFields.  This object in turn contains an object for each field to be validated.  The field objects within oFields have three properties.  The first is the description that will appear in a JavaScript message box if the field value is invalid.  The second is the type of validation to perform.  This type is case sensitive since it is used to call the corresponding JavaScript function.  For example, "String" will be used to call isString().  The third property of the field object is a boolean indication of whether this field is required.  If set to 1 the field cannot be left blank.  If set to 0 the field can be left blank but will be validated if populated.

It is sometimes necessary to add to and remove from the list of fields to be validated.  The user may select “store pickup” in which case it is no longer necessary to validate their shipping address.  If the oFields object has already been created then a field object can be added with, for example,

```
oFields.frmIfNotCovered = new field("Prescription","Radio",1);
```

This is a different type of object constructor so the properties are not named explicitly.  A field object may be removed with, for example,

```
delete oFields.frmUPSList;
```

These are standard JavaScript methods that can be studied further in the JavaScript documentation.

The file containing the validation functions must always be included as follows:

```
<script language="javascript" SRC="include/validation.js"></script>
```

It contains a master function, `isValid(sForm,oFields)`, that iterates through the field objects in the `oFields` object and calls the corresponding validation function.  If errors are found it combines then into a single error message that is displayed to the user through a JavaScript alert.  The `isValid()` function is usually called through the `onSubmit` event of the `<form>` tag.  The individual validation functions always return boolean.

## Active Server Page Design
The active server pages, perceived by the user as pharmacy forms, follow a common organizational scheme for program flow.  The top of all pages should include

```
<% Option Explicit %>
<% Response.Buffer = True %>
<!--#INCLUDE VIRTUAL="/shop/include/pharmacy_settings.inc"-->
<!--#INCLUDE VIRTUAL="/shop/include/pharmacy_functions.inc" -->
```

Line one requires that variables be declared explicitly and two increases performance by saving the screen write (HTML output) for after the completion of all code execution.  The `pharmacy_settings.inc` dimensions common variables, sets common values, includes all pharmacy constants, loads the appropriate style sheet and starts the page HTML.  Some of these will be discussed further, below.  The file `pharmacy_functions.inc` contains a number of VBScript functions that are used repeatedly throughout the site.  For example, formatting phone numbers and creating drop down lists from recordsets.

After dimensions unique variables, the first code on most pages is to check whether the page has been posted.  Most pages post to themselves.  Whether it has been posted can be evaluated by reading the value of a unique form field that will always be populated if the page was posted.  If posted, an array of field names is created to pass to the Blir VB.  We have written VBScript to automatically generate corresponding dummy arrays for the validation types and descriptive names for the form fields, no longer handled in the VB but still expected by the VB functions.  These three arrays are then passed to a VB function.  In some cases values are returned, such as a new receipt ID.  Finally, a `Response.Redirect` sends the user to the next page.

If the page has not been posted then the `sAction` parameter is read to determine if the form is for an add or edit of information.  If the former, default values are inserted in the form fields, usually empty strings.  If the latter, an edit, then a stored procedure is called to read the existing values from the database.  Those values are then assigned to the same variables that populate the form fields.

At the end of the VBScript block are the JavaScript includes, which may be as follows:

```
<script language="javascript" SRC="include/calpop.js"></script>
<script language="javascript" SRC="include/validation.js"></script>
<script language="javascript" SRC="include/toggle.js"></script>
<script language="javascript" src="include/cookies.js"></script>
<script language="javascript" src="include/pharmacy_flow.js"></script>
```

Some of these will be discussed in further detail.  Following these is typically a JavaScript block of code unique for this page.  Following that is normal HTML.

## Persistence
A handful of values must remain consistent across many or all of the pharmacy forms.  These include the shopper ID, patient ID, receipt ID, order ID, store number, fulfillment site ID, prescription type and payment type.  The shopper ID is passed to the pharmacy in a cookie set by the main Savon site.  This cookie is read and assigned to a VBScript variable within `pharmacy_sitesettings.inc`.  As developed by Blir, the rest of these values, and others, were also held in cookies.  After encountering many inconsistencies with cookies, these were transitioned to hidden form fields and query string parameters.  This resulted in improved reliability and speed.  These values are now read from the Request object and assigned to VBScript variables within `pharmacy_sitesettings.inc`.  The function `makeCommonFields()` in `pharmacy_functions.inc` creates a hidden form field for every value.  For buttons that will not use a form submission, but rather a link, the function `addCommonQS(ByVal sURL)` will append these same parameters to a URL.

## Flow
To prevent users from moving to pages out of sequence, by using their browser’s history or typing a URL directly, code was implemented to limit those pages that may refer to the page being loaded.  This posed a challenge since pages loaded from a browser’s history (e.g. using the “Back” button) are read from the client computer’s cache and not processed on the server (even with all the header options that supposedly prevent client caching).  This meant that server-side code could not be used.  JavaScript was the only solution.

Flow control is managed by a single include.  However, that include requires cookie management functions, so the following lines are required for flow control:

```
<script language="javascript" src="include/pharmacy_flow.js"></script>
<script language="javascript" src="include/cookies.js"></script>
```

Within the JavaScript of `pharmacy_flow.js` every page name is a variable containing a string list of valid referring pages.  If the name of the referring page, stored in a cookie, is found in the list then the page is loaded and its name stored in the cookie, otherwise JavaScript shows an alert and returns the user to the referring page.  Some situations are more complex.  There are times when a page may be allowed to refer to another, and other times when that same flow is disallowed.  These exceptions are documented in the code.
 
## Store Number / Pharmacy Validation
Store numbering, used to identify a single pharmacy, is in transition and so somewhat convoluted.  The table `tblFulfillmentSites` contains four fields that may contain the store number found on a user’s prescription label, though only `lAlbStoreNo` is used by related tables.  There are functions within `pharmacy_functions.inc` to ensure that a given store number is valid and to substitute the current number for one that has been retired.

## Refill Validation
Remainder redacted &hellip;





