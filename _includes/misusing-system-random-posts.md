{%- assign numbers = 
    "zero one two three four five six seven eight nine ten eleven twelve
    thirteen fourteen fifteen sixteen seventeen eighteen nineteen twenty" |
    split: ' ' -%}
{%- assign stats = site.data.misusing-system-random -%}
{%- assign instance-per-invocation = stats.instance.perInvocation | size -%}
{%- assign instance-per-thread     = stats.instance.perThread     | size -%}
{%- assign instance-singleton      = stats.instance.singleton     | size -%}
{%- assign arguments-good          = stats.arguments.good         | size -%}
{%- assign arguments-bad           = stats.arguments.bad          | size -%}
{%- assign seed-constant           = stats.seed.constant          | size -%}
{%- assign seed-poor               = stats.seed.poor              | size -%}
{%- assign seed-timing             = stats.seed.timing            | size -%}
{%- assign seed-good               = stats.seed.good              | size -%}
{%- assign seed-incorrect = seed-constant | plus: seed-poor | plus: seed-timing -%}
{%- assign divisor = stats.submissions | times: 1.0 -%}
{%- assign instance-per-invocation-pc = instance-per-invocation |
    divided_by: divisor | times: 100 | round -%}
{%- assign instance-singleton-pc = instance-singleton |
    divided_by: divisor | times: 100 | round -%}
{%- assign arguments-bad-pc = arguments-bad |
    divided_by: divisor | times: 100 | round -%}
{%- assign seed-incorrect-pc = seed-incorrect |
    divided_by: divisor | times: 100 | round -%}

{%- capture of-submissions -%}
of the {{ numbers[stats.submissions] }} submissions
{%- endcapture -%}
This is a series on common ways that `System.Random` is misused, based on
my experience reviewing code test submissions from job candidates.

{%- assign series_posts = site.posts |
    where: "title", "Misusing System.Random" |
    reverse -%}
{%- for post in series_posts %}
* [{{post.subtitle | default: post.title }}]({{ post.url }})
{%- endfor -%}