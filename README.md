# OmniAuth Instagram Business

Instagram Business OAuth2 Strategy for OmniAuth - basically it's Facebook OAuth2 Strategy (it just uses a different name :))

Supports OAuth 2.0 server-side and client-side flows. Read the Facebook docs for more details: http://developers.facebook.com/docs/authentication

## Installing

Add to your `Gemfile`:

```ruby
gem 'omniauth-instagram_business', github: 'paladinsoftware/omniauth-instagram_business'
```

Then `bundle install`.

## Usage

`OmniAuth::Strategies::InstagramBusiness` is simply a Rack middleware. Read the OmniAuth docs for detailed instructions: https://github.com/intridea/omniauth.

Here's a quick example, adding the middleware to a Rails app in `config/initializers/omniauth.rb`:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :instagram_business, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
```

[See the example Sinatra app for full examples](https://github.com/mkdynamic/omniauth-facebook/blob/master/example/config.ru) of both the server and client-side flows (including using the Facebook Javascript SDK).

## Configuring

You can configure several options, which you pass in to the `provider` method via a `Hash`:

Option name | Default | Explanation
--- | --- | ---
`scope` | `email` | A comma-separated list of permissions you want to request from the user. See the Facebook docs for a full list of available permissions: https://developers.facebook.com/docs/reference/login/
`display` | `page` | The display context to show the authentication page. Options are: `page`, `popup` and `touch`. Read the Facebook docs for more details: https://developers.facebook.com/docs/reference/dialogs/oauth/
`image_size` | `square` | Set the size for the returned image url in the auth hash. Valid options include `square` (50x50), `small` (50 pixels wide, variable height), `normal` (100 pixels wide, variable height), or `large` (about 200 pixels wide, variable height). Additionally, you can request a picture of a specific size by setting this option to a hash with `:width` and `:height` as keys. This will return an available profile picture closest to the requested size and requested aspect ratio. If only `:width` or `:height` is specified, we will return a picture whose width or height is closest to the requested size, respectively.
`info_fields` | 'name,email' | Specify exactly which fields should be returned when getting the user's info. Value should be a comma-separated string as per https://developers.facebook.com/docs/graph-api/reference/user/ (only `/me` endpoint).
`locale` |  | Specify locale which should be used when getting the user's info. Value should be locale string as per https://developers.facebook.com/docs/reference/api/locale/.
`auth_type` | | Optionally specifies the requested authentication features as a comma-separated list, as per https://developers.facebook.com/docs/facebook-login/reauthentication/. Valid values are `https` (checks for the presence of the secure cookie and asks for re-authentication if it is not present), and `reauthenticate` (asks the user to re-authenticate unconditionally). Use 'rerequest' when you want to request premissions. Default is `nil`.
`secure_image_url` | `false` | Set to `true` to use https for the avatar image url returned in the auth hash.
`callback_url` / `callback_path` | | Specify a custom callback URL used during the server-side flow. Note this must be allowed by your app configuration on Facebook (see 'Valid OAuth redirect URIs' under the 'Advanced' settings section in the configuration for your Facebook app for more details).

For example, to request `email`, `user_birthday` and `read_stream` permissions and display the authentication page in a popup window:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :instagram_business, ENV['APP_ID'], ENV['APP_SECRET'],
    scope: 'email,user_birthday,read_stream', display: 'popup'
end
```

### API Version

OmniAuth Facebook uses versioned API endpoints by default (current v2.6). You can configure a different version via `client_options` hash passed to `provider`, specifically you should change the version in the `site` and `authorize_url` parameters. For example, to change to v3.0 (assuming that exists):

```ruby
use OmniAuth::Builder do
  provider :instagram_business, ENV['APP_ID'], ENV['APP_SECRET'],
    client_options: {
      site: 'https://graph.facebook.com/v3.0',
      authorize_url: "https://www.facebook.com/v3.0/dialog/oauth"
    }
end
```

### Per-Request Options

If you want to set the `display` format, `auth_type`, or `scope` on a per-request basis, you can just pass it to the OmniAuth request phase URL, for example: `/auth/instagram_business?display=popup` or `/auth/instagram_business?scope=email`.

## Auth Hash

Here's an example *Auth Hash* available in `request.env['omniauth.auth']`:

```ruby
{
  provider: 'instagram_business',
  uid: '1234567',
  info: {
    email: 'joe@bloggs.com',
    name: 'Joe Bloggs',
    first_name: 'Joe',
    last_name: 'Bloggs',
    image: 'http://graph.facebook.com/1234567/picture?type=square',
    verified: true
  },
  credentials: {
    token: 'ABCDEF...', # OAuth 2.0 access_token, which you may wish to store
    expires_at: 1321747205, # when the access token expires (it always will)
    expires: true # this will always be true
  },
  extra: {
    raw_info: {
      id: '1234567',
      name: 'Joe Bloggs',
      first_name: 'Joe',
      last_name: 'Bloggs',
      link: 'http://www.facebook.com/jbloggs',
      username: 'jbloggs',
      location: { id: '123456789', name: 'Palo Alto, California' },
      gender: 'male',
      email: 'joe@bloggs.com',
      timezone: -8,
      locale: 'en_US',
      verified: true,
      updated_time: '2011-11-11T06:21:03+0000',
      # ...
    }
  }
}
```
