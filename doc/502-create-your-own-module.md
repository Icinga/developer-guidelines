
# Create your own module

## Where should I start?

Probably the most important question is usually what you want to do with his module. In our training we will first experiment with the given possibilities and then implement a small practical example.

## What should I name my module?

Once you know what the module is going to do, the hardest task is often choosing a good name. Ideally, it will tell you what the module actually does. But the name should not be too complicated, because we'll use it in PHP namespaces, directory names, and URLs.

Your own (company) name is often a good starting point. Our chosen module name for our first steps in training today will be `training`.

## Create and activate a new module

    mkdir -p /usr/local/icingaweb-modules/training
    icingacli module list installed
    icingacli module enable training

And done!
