# HTML Forms and Params

## Overview

In this walkthrough, we're going to be learning about connecting HTML forms to a Sinatra application by building a form that takes a user's name and favorite food (For example, the classic Dr. Seuss "Sam" whose favorite food evenutally becomes "Green Eggs and Ham"), and returns an interpolated string that combines the two ("My name is Sam, and I love Green Eggs and Ham"). 

## Objectives

1. Build a basic HTML form
2. Connect an HTML form to a sinatra application via the `method` and `action` attributes
3. Build a `POST` route in Sinatra's application controller to accept data from a form via params.
4. Understand that params are a Ruby hash containing form data.

![Green Eggs and Ham](https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcSwGoDVUDPLd8f_397Fp9BrLKlu-6msu0sSiT0LeEZQ8Ga2228z)

## Forms, Forms, Forms!

Think about how many forms you fill out online every day. Credit card payments, logins, registration forms, and even searches Google - all forms.

That's because form are the most common way for users to pass data to a web application. In this walkthrough, we'll create a new HTML form and connect it to a Sinatra application so that we can we can use and manipulate the data provided by the user.

## Instructions

If you want to code along, fork and clone this lesson. Within this lesson is a Sinatra application that you can use to code along.

Let's take a quick tour of the starter code. Open `app.rb`. The only route in our application responds to a `get` request to the `/food_form` URL by rendering the HTML in `food.erb`. We'll be working with `app.rb` (also called our Application Controller), and `food.erb` in the `views` directory. 

### Starting Your Application

To start this ruby web application, type `shotgun` in the root of this lesson's directory. 

#### Problems with Shotgun

*We can move this to it's own README and link to it. Given where this lesson is, I imagine we've already covered this content but just in case, I wrote it and we can remove it.*

If you see output that tells you that Shotgun/Thin or anything are listening on an IP address like `127.0.0.1` or `0.0.0.0` or localhost, everything worked. It should look something like:

```
// ♥ shotgun
== Shotgun/Thin on http://127.0.0.1:9393/
Thin web server (v1.6.3 codename Protein Powder)
Maximum connections set to 1024
Listening on 127.0.0.1:9393, CTRL+C to stop
```

You might get an error about `bundler`, it'll tell you to run `bundle install`, it'll look like this:

```
// ♥ shotgun
bundler: command not found: shotgun
Install missing gem executables with `bundle install`
```

Just run `bundle install` and try `shotgun` again.

You also might get an error about a port being in use. It'll look like this:

```
// ♥ shotgun
== Shotgun/Thin on http://127.0.0.1:9393/
Thin web server (v1.6.3 codename Protein Powder)
Maximum connections set to 1024
Listening on 127.0.0.1:9393, CTRL+C to stop
/Users/avi/.rvm/gems/ruby-2.2.2/gems/eventmachine-1.0.8/lib/eventmachine.rb:534:in `start_tcp_server': no acceptor (port is in use or requires root privileges) (RuntimeError)
```

That means you have another shotgun server running somewhere. Do you have another Terminal or Shell open running a web application or shotgun? You need to find that process or tab that is running a web application using shotgun and shut it down with `CTRL+C`.

### The Favorite Foods Form

Once your `shotgun` server is started, navigate to: [http://localhost:9393/food_form](http://127.0.0.1:9393/food_form).

You'll see a very basic HTML page. Your first task is to build a Form on this page.

#### Form Review

We're not going to dive too deep into form-making here, but the basics are:

+  HTML forms need a `<form>` opening tag and a `</form>` closing tag.

+ Each form field (text, date, password, etc) has a tag (usually `<input>`) with an attribute `type` denoting the type of form field. For example, a text box field looks like this:

```html
<input type="text">
```

+ A form needs a submit button:

```html
<input type="submit">
```

That's all a basic form needs! 

#### Our First Form

Let's build a form with two input fields, one for your name and one for your favorite food. We'll enclose our `<input>` tags in `<p>` tags so we can give them some textual context.

Put it all together and your HTML form will look like this:

**Copy the code below to `views/food_form.erb`:**

```html
<form>
  <p>Your Name: <input type="text"></p>
  <p>Your Favorite Food: <input type="text"></p>
  <input type="submit">
</form>
```

Now if you run `shotgun` and go to the corresponding view (`localhost:9393/food-form`, you'll see your very basic form.

### Connecting the form to your Sinatra App

If you try submitting the form, nothing will happen. That's because the form is not yet connected to our Application Controller in `app.rb`. There is nothing telling the form to send the user's data to our application.

In order to connect the form to our application, we need to give it expicit directions on **where** and **how** to send the data from the user. Both of these pieces of data are attributes that we give our `<form>` tag. 

```
<form method="POST" action="/food">
```

+ The `method` attribute tells the form what kind of request should be fired to the server when the submit button is clicked. In general, forms use POST request, because it is 'posting' data to the server.

+ The `action` attribute tells the form what specific route the post request should be sent to. In this case, we're posting to a route called `/food-receiver`

* Not enough about action routing. Before I would explain params and name I would make the route connect.*

Each form field `<input>` also must define a `name` attribute. The `name` attribute of an `<input>` defines how our application will identify each `<input>` data. 

All user submitted data will appear in a `params` hash accessible throughout our Sinatra controllers. The `name` attribute of an `<input>` corresponds to a key in the `params` hash for that data.

If you create a text field input with `<input type="text" name="favorite_food">`, whenever the user submits that form, you will be able to access the data entered into the Favorite Foods text box via `params[:favorite_food]`.

This is because we will be passing our data in the form of a hash, where the key will be the name of the data, and the value will be the data itself. In this case, we want our hash (which we call `params`) to look something like this:

```ruby
params = { 
  :name => "Sam",
	:favorite_food => "Green Eggs and Ham"
}
```

So our input names will need to be `name` and `favorite food`:

**Update the form in `views/food_form.erb` to:**

```html
<form method="POST" action="/foodreceiver">
  <p>Your Name:<input type="text" name="name"></p>
  <p>Your Favorite Food:<input type="text" name="favorite_food"></p>
  <input type="submit">
</form>
```

Let's see what happens when we submit this form...

![Sinatra Error](https://curriculum-content.s3.amazonaws.com/web-development/Sinatra/localhost_9393_foodreceiver.png)

We get a Sinatra error! This is great news. Sinatra errors tell us exactly what we need to do next to make the form work.

## Post Routes and Params

The error message Sinatra gives us is telling us that we don't have a route to receive the data from the HTML form that we created in `food_form.erb`. 

If you recall, we gave our form a method attribute of `POST` and an `action` attribute of `"/foodreceiver"`. Again, this is the **how** and **where** the data goes from this form.


Every form needs a corresponding route in the controller file (`app.rb`). Instead of a `get` route (which we used to route users to view an html page), we'll set up a post route:

```ruby
post '/foodreceiver' do

end
```

Notice that both of the attributes from the form are covered in this route: The `method` `post` and the `action` `/foodreceiver`. It's almost like a game of catch - the form is throwing the data to the server, which catches it by having the same receiving address (`/formreceiver`) and way of receiving the data (`post`).

### Param I Am

The data from the form comes nicely packaged up in the form of a hash called `params`. Let's set the return value of the post route to be params.to_s, and see what our form does now...

```
post '/foodreceiver' do
    params.to_s
  end
```

When you submit your form, you should now see the contents of `params` displayed as a hash in your browser like this:

```
{"name"=>"Sam", "favorite_food"=>"Green Eggs and Ham"}
```

Great! This means you've been able to successfully get the data from the form in to the controller, and can now manipulate it any way you want. 

Let's use the key-value pairs in `params` to return the following phrase, using good-old string interpolation:

```
"My name is #{params[:name}}, and I love #{params[:favorite_foods]}"
```

Here's the full post route and action in `app.rb`:

**Add this code to `app.rb`:**

```
post '/foodreceiver' do
  "My name is #{params[:name}}, and I love #{params[:favorite_foods]}"
end
```

Submit the form and see what happens! If you've gotten this far you can successfully connect an HTML form to your Sinatra app, and can use the params hash to use and manipulate data from the user. 
