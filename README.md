bootstrap_autocomplete_input
=============================

Autocomplete/typeahead input ready to be used with Bootstrap 4 in Rails 5.

For Bootstrap 3 and Rails 4 - see [bootstrap3_autocomplete_input gem](https://github.com/maxivak/bootstrap3_autocomplete_input).  

Features:
* Adds an input with autocomplete/typeahead compatible with Bootstrap 4.
* Works with [SimpleForm](https://github.com/plataformatec/simple_form).
* Uses the autocomplete library from [Bootstrap-3-Typeahead](https://github.com/bassjobsen/Bootstrap-3-Typeahead) that works with Bootstrap 4. 


# Install

## Gemfile

```ruby
gem 'simple_form'
gem 'bootstrap_autocomplete_input'

```

It assumes that you have Bootstrap 3 in your application. For example, you can use the [bootstrap-sass gem](https://github.com/twbs/bootstrap-sass).

```ruby
gem 'jquery-rails', '~>4.2.2'
gem 'jquery-ui-rails'
gem 'sass-rails', '~>5.0.6'

gem 'bootstrap', '~> 4.0.0.alpha6'


# Tooltips and popovers depend on tether for positioning. If you use them, add tether to the Gemfile:
source 'https://rails-assets.org' do
  gem 'rails-assets-tether', '>= 1.3.3'
end

# if problems with SSL - use source 'http://insecure.rails-assets.org' 

...

end
```


## Javascript

Include two js files (bootstrap3-typeahead.min, bootstrap-autocomplete-input.min) in your assets file 'app/assets/javascripts/application.js' after 'bootstrap.js'.

```bash
//= require jquery2
//= require jquery_ujs
//= require bootstrap
//= require bootstrap3-typeahead.min
//= require bootstrap-autocomplete-input.min

```


## CSS

All CSS for autocomplete input is already contained in Bootstrap 3 CSS.
  

For example, if you use bootstrap-sass, your css files 'app/assets/stylesheets/application.css.scss' might be like

```  
@import "bootstrap";
```




# Usage

Assuming that we have models Order and Client.

```ruby
class Order < ActiveRecord::Base
  belongs_to :client

end

class Client < ActiveRecord::Base
  has_many :orders

end

```


We will add autocomplete input to order's form for editing its client.


## controller

```ruby
class ClientsController < ApplicationController
   autocomplete :client, :name
   ...
end
```

This will generate the controller method 'autocomplete_client_name' that returns JSON data with clients.


## routes

Add routes for the generated controller action:

```ruby
resources :clients do
  get :autocomplete_client_name, :on => :collection
end
    
```

This will generate route with name `autocomplete_client_name_clients` which can be accesedby path autocomplete_client_name_clients_path.


## model

specify what to display as a value in a text field:

```ruby
class Client < ActiveRecord::Base
	belongs_to :order

	def to_s
		name
	end
end


```

## view

### simple_form

use :autocomplete input type for the client's field in 'app/views/orders/_form.html.haml'

```ruby
=simple_form_for @order do |f|
    = f.error_notification

    = f.input :field1
    
    = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url

    = f.button :submit, 'Save', :class=> 'btn-primary'
```



# Controller

The gem has method "autocomplete" to generate an action in your controller:

```ruby
class ClientsController < ApplicationController

  autocomplete :client, :name

```  
This will add new action 'autocomplete_client_name' where :client is the model class name and :name is the field name in the  model. The action uses params[:q] for the searching term and queries the database.

The action returns data in JSON format:
```text
[["id1", "name1"], ["id2", "name2"], ..]
```
which is an array of ids and titles.


## Options for controller method

```ruby
class ClientsController < ApplicationController

  autocomplete :client, :name, { options_hash_here }

```  


### class_name



### column_name

```ruby
class ClientsController < ApplicationController

  autocomplete :client, :name, { :column_name => 'title_long' }

```  

It will search in this column in database.

### extra_columns

```
class ClientsController < ApplicationController

  autocomplete :client, :name, { extra_columns: [:lastname, :birthdate], column_name: 'fullname' }

```


### display_id

* Method used for getting ID of the record. 
* Make sure that columns this method depends on are included in :extra_columns option.


### display_value

```ruby
class ClientsController < ApplicationController

  autocomplete :client, :name, { :display_value => 'fullname', :full_model=>true }
  ...
end
```

### where

* additional WHERE conditions


# model

class Client < ActiveRecord::Base

  def name_with_birthdate
    firstname+', '+birthdate
  end
end

```  

It will display search results using this method. You must set option :full_model=>true if you search by one column but you want to display value from other columns.

```ruby
# JSON data returned by the controller
[[1, 'Mark Twen, 1980'], [2, 'John Deer, 1985'], ..]

```


# Options for input

specify options:

```ruby
= simple_form_for @order do |f|

  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url, :option_name => opt_value, :option_name2=>value2

```


## Data source

You have several options to get data for search:
 - :source => url - get static data downloaded from URL
 - :source_query => url - query data with a keyword from URL
 - :source_array => array - data is stored in local array of strings


examples:
```ruby

  = f.input :client, :as => :autocomplete, :source => autocomplete_client_name_clients_url
  
  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url
  
  = f.input :client, :as => :autocomplete, :source_array => ['item1', 'item2', ..]
```




## Object id

Autocomplete input adds a hidden field to use the id of the selected item.
Unless you use local array as data source (:source_array), the hidden field for the id is added automatically.

```ruby
  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url
```

This will generate HTML like
```html
<input id="order_client_id" type="hidden" value="0" name="order[client_id]">

<input id="order_client" name="order[client]"
class="autocomplete optional" type="text" value="*some value*" 
data-source-query="http://localhost:3000/clients/autocomplete_client_name" data-provide="typeahead" data-field-id="order_client_id" 
autocomplete="off">

```

This will update the field with id "order_client_id" with the id of the selected object. 

Data posted from the form to the server will contain the id of the selected client (params[:order][:client_id])
```text
params[:order]

{
client_id: 5,
client: 'client name',
...
}
```




If you don't want to have a hidden field for object id, set :field_id option to false:

```ruby
  = f.input :client, :as => :autocomplete, :source => autocomplete_client_name_clients_url, :field_id=>false
```

This will generate HTML without a hidden field:
```html
<input id="order_client" name="order[client]"
class="autocomplete optional" type="text" value="*value*" 
data-source="http://localhost:3000/clients/autocomplete_client_name" 
data-provide="typeahead" 
autocomplete="off">

```



## :items

The max number of items to display in the dropdown. Default is 8.
```ruby
= simple_form_for @order do |f|

  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url, :items => 2

```



## :minLength
The minimum character length needed before triggering autocomplete suggestions. You can set it to 0 so suggestion are shown even when there is no text when lookup function is called. Default is 1.


```ruby
= simple_form_for @order do |f|

  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url, :minLength=>3

```



View the full list of options at https://github.com/bassjobsen/Bootstrap-3-Typeahead. Note that not all options from that list are supported by the current gem.


## afterSelect

The callback function to be execute after an item is selected.

```ruby
= simple_form_for @order do |f|

  = f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url, :minLength=>3, :afterSelect=>'after_select_client'




:javascript
  function after_select_client(item){
    console.log('item is selected');
    console.log(item);

  }


```




# Examples

## Static data from the server

```ruby
= simple_form_for @order, html: { autocomplete: 'off' } do |f|
  -#.. other fields..
  
  = f.input :client, :as => :autocomplete, :source => autocomplete_client_name_clients_url
  
```

Data will be loaded from the server once after page loads, and all queries to search a string will be against the stored data.
Do not use this if you have a big number of items for client.
    

## Query data from the server


```ruby
= f.input :client, :as => :autocomplete, :source_query => autocomplete_client_name_clients_url
```

Data will loaded from the server every time you type a new string in the field.


## Search by name, display full name in search result

```ruby
class ClientsController < ApplicationController

  autocomplete :client, :name, { :display_value => 'name_with_birthdate', :full_model=>true }
  ...
end


# model

class Client < ActiveRecord::Base

  def name_with_birthdate
    firstname+', '+birthdate
  end
end




```


# Tests

# Development 

This gem uses the ORM library for ActiveRecord models included in [rails3-jquery-autocomplete](https://github.com/peterwillcn/rails4-autocomplete/tree/master/test/lib/rails3-jquery-autocomplete)


# TODO

Write tests




