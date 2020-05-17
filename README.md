# Sami
A javascript library that presents common html and form controls as a simple programmable model.  While it does come with some generally useful utility functions, its main features are:
* functional HTML rendering
* form rendering and data handling based on configurable data models

### Dependencies

Sami depends on:
* jQuery (most versions), though it only uses a few features (events, lookups, add/remove css class, hide/show)
* faicon() function requires inclusion of Font Awesome CSS (packaged with 4.7.0 but the version can be changed)
* 'markdown' form control requires 'markdown'

Not formally a jQuery plug-in.

### How the library got its name

![alt text](https://github.com/h3rb/Sami/raw/master/Reindeer_and_pack,_with_Lapp_driver.jpg "Pack reindeer with Sami Driver from The land of the midnight sun, c. 1881")

The Saami people (also Sámi or Sami) are a Finno-Ugric people inhabiting Sápmi, which today encompasses large northern parts of Norway, Sweden, Finland and the Kola Peninsul within the Murmansk Oblast of Russia.  In English, they have been historically known as Lapps or Laplanders.  Among other livelihoods, the Sámi people's best-known livelihood was as semi-nomadic reindeer herders. 

The reason this library is named Sami is to conceptualize the idea of herding data from a client form into an API endpoint.  Other ideas for names were htCamel, SamiForm (a play on semaphore) and PackForm.

### Implementation Philosophy

Sami is implemented functionally on top of vanilla Javascript.  Instead of implementing Sami as an object, it is a series of "free functions", but it is possible to import Sami as an object to avoid global namespace collisions or to modularize/compartmentalize it.

Sami creates a dialect that handles:
* Basic HTML5 tags
* Forms with common controls like sliders, text input, text areas, markdown input
* Advanced forms with hide/reveal and/or extendable sections with recursive functionality
* Form data acquisition into a JSON object

## Uses and Workflows

### Simple, Functional HTML Rendering

Sami's HTML rendering functions are named after HTML tags (with few exceptions).  They can be used to generate HTML with common elements, or extended to include uncommon element names (user-defined).

Take the case of needing to replace the .innerHTML of an element.  A Javascript program often enters into a complex syntactical acrobatics to render such elements.

Consider this code below, that programattically populates a drop-down menu in a bootstrap application (from https://adminlte.io) from a datasource:

```javascript
// Example of how code might be structured without Sami HTML helpers
var element=document.getElementById("drop-down-7");
var options=
[
  // Option 1
  '<a href="#"><div class="pull-left">'
  +'<img src="img/user2-160x160.jpg" class="img-circle" alt="User Image">'
  +'</div><h4>Support Team'
  +'<small><i class="fa fa-clock-o"></i> '+datasource[1].minutes+' mins</small>'
  +'</h4><p>Why not buy a new awesome theme?</p></a>',
  // Option 2
  '<a href="#"><div class="pull-left">'
  +'<img src="img/user3-128x128.jpg" class="img-circle" alt="User Image">'
  +'</div><h4>AdminLTE Design Team<small><i class="fa fa-clock-o"></i> '+datasource[2].hours+' hours</small>'
  +'</h4><p>Why not buy a new awesome theme?</p></a>'
];
var output='<ul class="menu">';
for ( var i=0; i<options.length; i++ ) {
 output+='<li>';
 output+=options[i];
 output+='</li>';
}
output+='</ul>';
element.innerHTML=output;
```

While the above example syntatically works (the data is populated from a "datasource"), it illustrates the problematic maintenance of a fairly simple set of HTML tags and programatically populating them with data.  More complex trees of tags and the difficulty in figuring out where tags begin and end in common syntax highlighters pose a problem, since they are generally working with a single language in a .js file, Javascript, though some highlighters are "smart enough" to recognize the embedded HTML.

In Sami, the same code might look like this:

```javascript
// Example refactored with Sami's functional HTML helpers:
 var element=Get("someting");
 var options=[
   hrefbtn( 
    div( img( "img/user3-128x128.jpg", "img-circle" ), null, "pull-left" )
    +h4("Support Team" + small(faicon("fa-clock-o")+nbsp(1)+datasource[1].timestring) )
    +p("Why not buy an awesome theme?"),
    "myApp.DoItFunction(this,1)"
   ),
   hrefbtn( 
    div( img( "img/user3-128x128.jpg", "img-circle" ), null, "pull-left" )
    +h4("Option 2" + small(faicon("fa-clock-o")+nbsp(1)+datasource[2].timestring) )
    +p("Why not buy an awesome theme?"),
    "myApp.DoItFunction(this,1)"
   )
 ];
var output='';
for ( var i=0; i<options.length; i++ ) output+=li(options[i]);
element.innerHTML=ul(output,"menu");
```

The above code is easier to manage and, once structured, can be easily refactored as a functional "block" rather than as a gangly difficult-to-manage syntatical nightmarish string.

### Form handling helpers

Sami can be used to both render a form and to handle its data.  It does not perform validation (it's assumed to happen server-side), but can be extended to do so since the "DOM ids" that it generates are predictable and user-configurable.   Forms can be placed anywhere.  It work swith jQuery, Angular, Bootstrap, you name it.

In Javascript (this example also uses jQuery), a Javascript app form might look like:
```javascript
element.innerHTML='<div><label for="inputtag">My Label</label><input type="text" id="inputtag" name="inputtag" placeholder="something..." value="'
                 +myData.existingInputTagValue
                 +'" data-storage="somespecialtag" /><button id="submitform"><i class="fa fa-plus"></i></button></div>';
$("#submitform").click(function(e){
 var value=$("#inputtag").val();
 /* do something with value */
});
```

Sami takes a different approach that keeps things more consistent and easier to manage.  

It assumes firstly:
1. Data may originate with a user, but is ultimately going to be stored in a structured object.  
2. Once data is in a structured object, it may need to be edited later.
3. The structure of the edit-able data is almost exactly the same as the way it was inputted.

Therefore, data adheres to a model.

The two basic assumptions are two-thirds of the Edit-Add-View design pattern.  This usually occurs in the order of "Add", "View", "Edit", and for the purposes of rapid development, the "View" portion may be similar to the "Edit" form, except read-only, unless something specific needs to be done with the inputs.  However, Edit and View are almost always the same for structured data that has been added by the user.

Secondly, it assumes that you will be using styled HTML form elements, generally.  While you can extend Sami to use all sorts of off-spec components, its support for those basic tried-and-true elements is core to its form handling functionality.
