# Routes

For this example, assume we are using the following schema:

* Bands have a name, and have many albums
* Albums have a title and year, and belong to bands

And we will describing these routes dealing with *creating new albums*.

## nested `:new`

If you have a nested new, creating an album will have it's own view accessible with `GET /bands/:band_id/albums/new`

### routes

```ruby
resources :bands do
  resources :albums, only: [:new]
end

resources :albums, except: [:new]
```

### controller

```ruby
class AlbumsController < ApplicationController
  def create
    album = Album.new(album_params)
    if album.save
      redirect_to band_url(album.band_id)
    else
      flash.now[:errors] = album.errors.full_messages
      render :new
    end
  end

  def album_params
    params.require(:album).permit(:title, :year)
  end
end
```


### views

```html
<form action="<%= albums_url %>" method="POST">
  <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

  <!-- pass the band id we have from params (it's in the route path as a wildcard) so "create" action will have a band_id for the ablum we are creating -->
  <input type="hidden" name="album[band_id]" value="<%= params[:band_id] %>">

  <input type="text" name="album[title]">
  <input type="text" name="album[year]">
  <input type="submit" value="Create New Album">
</form>
```


## nested `:create`

If you have a nested create, you will not have a `new` view for creating albums. Instead, the form will live on the posts `show` view.

Our route for hitting `AlbumsController#create` will look like:

`POST /bands/:band_id/albums`

### route

```ruby
resources :bands do
  resources :albums, only: [:create]
end

resources :albums, except: [:create]
```

### controller

```ruby
class AlbumsController < ApplicationController
  def create
    album = Album.new(album_params)
    if album.save
      redirect_to band_url(album.band_id)
    else
      flash.now[:errors] = album.errors.full_messages
      render 'bands/show'
    end
  end

  def album_params
    params.require(:album).permit(:title, :year)
  end
end
```

### views

Because your form for creating an album is on the band show page `/bands/:id`, you will want to send that band id as part of the request you make when sending the form.

```html

<!-- to hit a nested create, the url helper method will be some combination of the two entities you are dealing with (check your routes for this!) -->
<form action="<%= band_albums_url %>" method="POST">
  <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

  <!-- pass the band id we have from params (it's in the route path as a wildcard) so "create" action will have a band_id for the ablum we are creating -->
  <input type="hidden" name="album[band_id]" value="<%= params[:id] %>">

  <input type="text" name="album[title]">
  <input type="text" name="album[year]">
  <input type="submit" value="Create New Album">
</form>
```


## not nested