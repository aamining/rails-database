# README

## Screenshots

![alt tag](https://raw.githubusercontent.com/ltfschoen/demo_fileupload/master/screenshots/screenshot_psql.png)
![alt tag](https://raw.githubusercontent.com/ltfschoen/demo_fileupload/master/screenshots/screenshow_view_fileuploaded.png)

## Setup Instructions

* Create Rails app specifically using PSQL DB
  ```
  rails new carrierwave-test --database=postgresql
  ```

* Generate Scaffold of Menu with name and image for File Upload
  ```
  rails generate scaffold Menu name:string image:text
  ```

* Start PSQL
* Create Rails app DB and migrate
  ```
  rails db:create db:migrate RAILS_ENV=development
  ```

* Setup File Upload
  * References:
    * https://medium.com/@mauddev/rails-5-and-carrierwave-53960ec20c4b

  * Add Carrierwave Gem for File Upload
    ```
    gem 'carrierwave', '~> 1.0'

  * bundleinstall

    rails generate uploader Avatar
    ```
    * Load Carrierwave after ORM load in application.rb (after Bundler.require) in congig folder
      ```
      require 'carrierwave/orm/activerecord'
      ```

    * Add column to Menus
      ```
      rails g migration add_avatar_to_menus avatar:string
      rails db:migrate
      ```
    * Add inside Menus Model (in Model Folder)
      ```
      mount_uploader :avatar, AvatarUploader
      ```

    * Modified the New Menu Form to show the File Upload
      ```
      <div class="field">
        <%= form.label :avatar %>
        <%= file_field(:menu, :avatar, accept: 'image/png,image/gif,image/jpeg') %>
      </div>
      ```
    * Check in PSQL
      ```
      rails db
      select * from menus;
      \q
      ```
    * Expore different Methods generated by Carrierwave
    (i.e. since we called the Menu class attribute `avatar`, it automatically
      generated a method `@menu.avator_url`)
      ```
      puts @menu.methods
      ```
    * Add to the Show view of Menu (i.e. views/menus/show.html.erb)
      ```
      <p>
        <strong>Avatar:</strong>
        <%= image_tag(@menu.avatar_url.to_s, class: "avatar_icon") %>
      </p>
      ```
    * Add CSS style to assets/stylesheets/menus.scss
      ```
      .avatar_icon {
        width: 100px;
        height: 100px;
      }
      ```
    * Inspect the Rails server logs that show the `menu_params` that are sent via POST request to
    the MenusController's `create` Action, which shows a temporary file stored on the local machine stores the image and the `filename` is associated with it and stored in the Menu table's `avatar` column as `bg_black.jpg`, i.e.
      ```
      Processing by MenusController#create as HTML
      Parameters: {"utf8"=>"✓", "authenticity_token"=>"9d2Y3YeYxHEde0m9OXIoqbpYDktl4f8zRtp+U8zH+Yvg3ayYeZQEfvtf91auKRC34Bl38Ib0XcskFnFHu5QuTg==", "menu"=>{"name"=>"black", "image"=>"", "avatar"=>#<ActionDispatch::Http::UploadedFile:0x007fd442819778 @tempfile=#<Tempfile:/var/folders/8q/kll0jy354kj5gxrv9n0pwx300000gn/T/RackMultipart20170915-64099-o9laf2.jpg>, @original_filename="bg_black.jpg", @content_type="image/jpeg", @headers="Content-Disposition: form-data; name=\"menu[avatar]\"; filename=\"bg_black.jpg\"\r\nContent-Type: image/jpeg\r\n">}, "commit"=>"Create Menu"}
      ```
    * Add the following to Menu Controller `new` Action to see what `@menu.avatar_url` is:
      ```
      puts @menu.avatar_url
      ```
      * Output is for example `/uploads/menu/avatar/2/bg_black.jpg`.
      Now if you change the browser URL to
      `http://localhost:3000/uploads/menu/avatar/2/bg_black.jpg`
      it loads your image.