# vkontakte_api

`vkontakte_api` is a Ruby wrapper for VKontakte API. It allows you to call all API methods in the simplest possible way.

Below is the English version of the readme. Russian version is located [here](http://github.com/7even/vkontakte_api/README.ru.md).

## Installation

``` bash
gem install vkontakte_api
```

## Usage

Default HTTP request library is `Net::HTTP`. You can set any other adapter supported by `faraday` in the configure block like so:

``` ruby
VkontakteApi.configure do |config|
  config.adapter = :net_http
end
```

All requests are sent via `VkontakteApi::Client` instance. To create one you just pass an access token.

``` ruby
@app = VkontakteApi::Client.new('my_access_token')
```

Probably the easiest way to get an access token in a web-app is using [OmniAuth](https://github.com/intridea/omniauth), but you can roll your own authentication if for some reason `OmniAuth` isn't acceptable. Currently `vkontakte_api` doesn't offer authentication functionality.

Now you can call the API methods. All method names are underscore_cased as opposed to the [official documentation](http://vk.com/developers.php?o=-1&p=%D0%9E%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5_%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%D0%BE%D0%B2_API) where they are camelCased, so `getGroups` becomes `get_groups`. You can still call them in a camelCased manner, but that is not a Ruby-way practice.

``` ruby
@app.get_user_settings  # => 327710
@app.groups.get         # => [1, 31022447]
```

Predicate methods (those which begin with `is`, like `is_app_user`) are expected to return the result of some condition, so the '?' is appended to the end of the method name, and they return `true` or `false`:

``` ruby
@app.is_app_user? # => true
```

You can still call these without a '?' and get `'1'` or `'0'` in result if you want to handle that yourself.

Now for parameters. All method parameters are named, and you can pass them in a hash where parameter names are keys and parameter values are values:

``` ruby
@app.friends.get(fields: 'uid,first_name,last_name')
# => [
  {
    :uid          => "1",
    :first_name   => "Павел",
    :last_name    => "Дуров"
  },
  {
    :uid          => "6492",
    :first_name   => "Andrew",
    :last_name    => "Rogozov"
  }
]
```

It's also worth noting that all returned hashes have symbolized keys.

If the response is an Enumerable, you can pass a block that will yield each successive element and put the result in the returned array (like `Enumerable#map` does):

``` ruby
@app.friends.get(fields: 'first_name,last_name') do |friend|
  "#{friend[:first_name]} #{friend[:last_name]}"
end
# => ["Павел Дуров", "Andrew Rogozov"]
```

`vkontakte_api` doesn't keep any method names list inside (except method namespaces like `friends` or `groups`) - when you call a method, it's name is camelcased before sending a request to VKontakte. So when a new method is added to the API, you don't need to wait for the new version of `vkontakte_api` gem - you can use that new method immediately. If you mistype the method name or use a method you don't have rights to call, you will get an exception with a relevant message (more about the exceptions below).

### Error handling

If VKontakte returns an error in response, you get a `VkontakteApi::Error` exception with all meaningful information that can be retrieved:

``` ruby
@app.audio.get_by_id
# => VkontakteApi::Error: VKontakte returned an error 1: 'Unknown error occured' after calling method 'audio.getById' with parameters {}.
```

## Roadmap

* Authentication (getting the access_token from VK)
* RDoc coverage

## Contributing

If you want to contribute to the project, fork the repository, push your changes to a topic branch and send me a pull request.

`vkontakte_api` is tested under MRI `1.8.7`, `1.9.2` and `1.9.3`. If something is working incorrectly or not working at all in one of these environments, this should be considered a bug. Any bug reports are welcome at Github issues.