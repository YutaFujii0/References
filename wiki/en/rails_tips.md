
# useful gems
- `devise` for user authentication [github](https://github.com/plataformatec/devise)
- `cancancan` for role distribution [github](https://github.com/CanCanCommunity/cancancan)
- `virtus` for type defining for non-ActiveRecord model [github](https://github.com/solnic/virtus)

# file architecture tips
- asset pipeline for webpacker
- use service objects
- use decorators
- what should be inside `lib` directory

# method tips
- safety operator `method1&.method2`
- similar but a little slower method named `object.try(method2)`

- use `presence` instead of `object.present? ? object : nil`
https://andycroll.com/ruby/use-the-presence-method/