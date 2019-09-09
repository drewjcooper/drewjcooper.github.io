This is a series on common ways that `System.Random` is misused, based on
my experience reviewing code test submissions from job candidates.

{% assign series_posts = site.posts |
    where: "title", "Misusing System.Random" |
    reverse %}
{% for post in series_posts
%}* [{% if post.subtitle %}{{post.subtitle}}{% else %}{{ post.title }}{% endif %}]({{ post.url }})
{% endfor %}