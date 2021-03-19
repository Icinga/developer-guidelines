Although it is not directly related to our topic, one thing stands out: our table doesn't exactly look very flashy (yet). To improve its looks, we can easily put CSS to our module. We create a suitable directory, the nameing follows the same scheme as before:

    mkdir public/css

We then add our CSS instructions in the file `module.less`. Less is a CSS extension, which adds a variety of functions, more can be found under [lesscss.org](http://lesscss.org/functions/). LESS also accepts conventional CSS. The nice thing about Icinga Web is, that there is no need to worry about whether ones CSS will influence other modules' - or Icinga Webs itself.

So we can easily define the following, without 'breaking' foreign tables:

    table {
        width: 100%;
    }

    th {
        width: 20%;
        text-align: right;
        line-height: 2em;
        padding-right: 2em;
    }

When we watch the requests in our browser's developer tools, we can see that Icinga Web loads css/icings.min.css as the only CSS file. We can also load css/icinga.css to conveniently view what Icinga Web has turned CSS code into:

    .icinga-module.module-training table {
      width: 100%;
    }
    .icinga-module.module-training th {
      width: 20%;
      text-align: right;
      line-height: 2em;
      padding-right: 2em;
    }

Prefixes ensure that our CSS only applies to the containers, in which our module displays its contents.

## Useful CSS classes

Icinga Web 2 provides a set of CSS classes that make our job easier. The class `common-table` is used for our default, list like, tables, `name-value-table` for name/value pairs where the identifier is displayed as `th` on the left and the corresponding value in a `td` on the right. Also useful is `table-row-selectable` - this changes the behavior of the table. The whole line is highlighted when you hover over it. If you click somewhere, the first link of the line will be considered clicked. In combination with `common-table`, the table looks a lot better already, without any additional styling.
