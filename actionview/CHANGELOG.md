*   Stop generating `Link preload` headers once it has reached 1KB.

    Some proxies have trouble handling large headers, but more importantly preload links
    have diminishing returns so it's preferable not to go overboard with them.

    If tighter control is needed, it's recommended to disable automatic generation of preloads
    and to generate them manually from the controller or from a middleware.

    *Jean Boussier*

*   `simple_format` helper now handles a `:sanitize_options` - any extra options you want appending to the sanitize.

    Before:
    ```ruby
      simple_format("<a target=\"_blank\" href=\"http://example.com\">Continue</a>")
      # => "<p><a href=\"http://example.com\">Continue</a></p>"
    ```

    After:
    ```ruby
      simple_format("<a target=\"_blank\" href=\"http://example.com\">Continue</a>", {}, { sanitize_options: { attributes: %w[target href] } })
      # => "<p><a target=\"_blank\" href=\"http://example.com\">Continue</a></p>"
    ```

    *Andrei Andriichuk*

*   Add support for HTML5 standards-compliant sanitizers, and default to `Rails::HTML5::Sanitizer`
    in the Rails 7.1 configuration if it is supported.

    Action View's HTML sanitizers can be configured by setting
    `config.action_view.sanitizer_vendor`. Supported values are `Rails::HTML4::Sanitizer` or
    `Rails::HTML5::Sanitizer`.

    The Rails 7.1 configuration will set this to `Rails::HTML5::Sanitizer` when it is supported, and
    fall back to `Rails::HTML4::Sanitizer`. Previous configurations default to
    `Rails::HTML4::Sanitizer`.

    *Mike Dalessio*

*   Add support for the HTML picture tag. It supports passing a String, an Array or a Block.
    Supports passing properties directly to the img tag via the `:image` key.
    Since the picture tag requires an img tag, the last element you provide will be used for the img tag.
    For complete control over the picture tag, a block can be passed, which will populate the contents of the tag accordingly.

    Can be used like this for a single source:
    ```erb
    <%= picture_tag("picture.webp") %>
    ```
    which will generate the following:
    ```html
    <picture>
        <img src="/images/picture.webp" />
    </picture>
    ```

    For multiple sources:
    ```erb
    <%= picture_tag("picture.webp", "picture.png", :class => "mt-2", :image => { alt: "Image", class: "responsive-img" }) %>
    ```
    will generate:
    ```html
    <picture class="mt-2">
        <source srcset="/images/picture.webp" />
        <source srcset="/images/picture.png" />
        <img alt="Image" class="responsive-img" src="/images/picture.png" />
    </picture>
    ```

    Full control via a block:
    ```erb
    <%= picture_tag(:class => "my-class") do %>
        <%= tag(:source, :srcset => image_path("picture.webp")) %>
        <%= tag(:source, :srcset => image_path("picture.png")) %>
        <%= image_tag("picture.png", :alt => "Image") %>
    <% end %>
    ```
    will generate:
    ```html
    <picture class="my-class">
        <source srcset="/images/picture.webp" />
        <source srcset="/images/picture.png" />
        <img alt="Image" src="/images/picture.png" />
    </picture>
    ```

    *Juan Pablo Balarini*

*   Remove deprecated support to passing instance variables as locals to partials.

    *Rafael Mendonça França*

*   Remove deprecated constant `ActionView::Path`.

    *Rafael Mendonça França*

*   Guard `token_list` calls from escaping HTML too often

    *Sean Doyle*

*   `select` can now be called with a single hash containing options and some HTML options

    Previously this would not work as expected:

    ```erb
    <%= select :post, :author, authors, required: true %>
    ```

    Instead you needed to do this:

    ```erb
    <%= select :post, :author, authors, {}, required: true %>
    ```

    Now, either form is accepted, for the following HTML attributes: `required`, `multiple`, `size`.

    *Alex Ghiculescu*

*   Datetime form helpers (`time_field`, `date_field`, `datetime_field`, `week_field`, `month_field`) now accept an instance of Time/Date/DateTime as `:value` option.

    Before:
    ```erb
    <%= form.datetime_field :written_at, value: Time.current.strftime("%Y-%m-%dT%T") %>
    ```

    After:
    ```erb
    <%= form.datetime_field :written_at, value: Time.current %>
    ```

    *Andrey Samsonov*

*   Choices of `select` can optionally contain html attributes as the last element
    of the child arrays when using grouped/nested collections

    ```erb
    <%= form.select :foo, [["North America", [["United States","US"],["Canada","CA"]], { disabled: "disabled" }]] %>
    # => <select><optgroup label="North America" disabled="disabled"><option value="US">United States</option><option value="CA">Canada</option></optgroup></select>
    ```

    *Chris Gunther*

*   `check_box_tag` and `radio_button_tag` now accept `checked` as a keyword argument

    This is to make the API more consistent with the `FormHelper` variants. You can now
    provide `checked` as a positional or keyword argument:

    ```erb
    = check_box_tag "admin", "1", false
    = check_box_tag "admin", "1", checked: false

    = radio_button_tag 'favorite_color', 'maroon', false
    = radio_button_tag 'favorite_color', 'maroon', checked: false
    ```

    *Alex Ghiculescu*

*   Allow passing a class to `dom_id`.
    You no longer need to call `new` when passing a class to `dom_id`.
    This makes `dom_id` behave like `dom_class` in this regard.
    Apart from saving a few keystrokes, it prevents Ruby from needing
    to instantiate a whole new object just to generate a string.

    Before:
    ```ruby
    dom_id(Post) # => NoMethodError: undefined method `to_key' for Post:Class
    ```

    After:
    ```ruby
    dom_id(Post) # => "new_post"
    ```

    *Goulven Champenois*

*   Report `:locals` as part of the data returned by ActionView render instrumentation.

    Before:
    ```ruby
    {
    identifier: "/Users/adam/projects/notifications/app/views/posts/index.html.erb",
    layout: "layouts/application"
    }
    ```

    After:
    ```ruby
    {
    identifier: "/Users/adam/projects/notifications/app/views/posts/index.html.erb",
    layout: "layouts/application",
    locals: {foo: "bar"}
    }
    ```

    *Aaron Gough*

*   Strip `break_sequence` at the end of `word_wrap`.

    This fixes a bug where `word_wrap` didn't properly strip off break sequences that had printable characters.

    For example, compare the outputs of this template:

    ```erb
    # <%= word_wrap("11 22\n33 44", line_width: 2, break_sequence: "\n# ") %>
    ```

    Before:

    ```
    # 11
    # 22
    #
    # 33
    # 44
    #
    ```

    After:

    ```
    # 11
    # 22
    # 33
    # 44
    ```

    *Max Chernyak*

*   Allow templates to set strict `locals`.

    By default, templates will accept any `locals` as keyword arguments. To define what `locals` a template accepts, add a `locals` magic comment:

    ```erb
    <%# locals: (message:) -%>
    <%= message %>
    ```

    Default values can also be provided:

    ```erb
    <%# locals: (message: "Hello, world!") -%>
    <%= message %>
    ```

    Or `locals` can be disabled entirely:

    ```erb
    <%# locals: () %>
    ```

    *Joel Hawksley*

*   Add `include_seconds` option for `datetime_local_field`

    This allows to omit seconds part in the input field, by passing `include_seconds: false`

    *Wojciech Wnętrzak*

*   Guard against `ActionView::Helpers::FormTagHelper#field_name` calls with nil
    `object_name` arguments. For example:

    ```erb
    <%= fields do |f| %>
      <%= f.field_name :body %>
    <% end %>
    ```

    *Sean Doyle*

*   Strings returned from `strip_tags` are correctly tagged `html_safe?`

    Because these strings contain no HTML elements and the basic entities are escaped, they are safe
    to be included as-is as PCDATA in HTML content. Tagging them as html-safe avoids double-escaping
    entities when being concatenated to a SafeBuffer during rendering.

    Fixes [rails/rails-html-sanitizer#124](https://github.com/rails/rails-html-sanitizer/issues/124)

    *Mike Dalessio*

*   Move `convert_to_model` call from `form_for` into `form_with`

    Now that `form_for` is implemented in terms of `form_with`, remove the
    `convert_to_model` call from `form_for`.

    *Sean Doyle*

*   Fix and add protections for XSS in `ActionView::Helpers` and `ERB::Util`.

    Escape dangerous characters in names of tags and names of attributes in the
    tag helpers, following the XML specification. Rename the option
    `:escape_attributes` to `:escape`, to simplify by applying the option to the
    whole tag.

    *Álvaro Martín Fraguas*

*   Extend audio_tag and video_tag to accept Active Storage attachments.

    Now it's possible to write

    ```ruby
    audio_tag(user.audio_file)
    video_tag(user.video_file)
    ```

    Instead of

    ```ruby
    audio_tag(polymorphic_path(user.audio_file))
    video_tag(polymorphic_path(user.video_file))
    ```

    `image_tag` already supported that, so this follows the same pattern.

    *Matheus Richard*

*   Ensure models passed to `form_for` attempt to call `to_model`.

    *Sean Doyle*

Please check [7-0-stable](https://github.com/rails/rails/blob/7-0-stable/actionview/CHANGELOG.md) for previous changes.
