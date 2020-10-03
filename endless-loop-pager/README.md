# An infinite loop Pager component with Lifecycle Events!

![Preview](preview.gif)

This is an example of a pager that enables a series of commands to be executed when entering (pageDidAppear) or leaving (pageWillDisappear) a page.

## How does it work?

First, various properties are bound to the pager, the pages from our data sources, the current page and the number of pages in the pager. Furthermore 2 APL event handlers "onMount" and "onPageChanged". In the onMount handler the commands defined in the datasource for the current page are executed.

> It is very important to add a sequencer to prevent our commands from being executed in fast mode.

```
"onMount": [
  {
    "type": "Sequential",
    "sequencer": "auto",
    "commands": [
      "${pagerItems[page].pageDidAppear}"
    ]
  }
]
```

In the onPageChanged handler we update the page property of our pager and execute again the commands defined for the current page.

```
"onPageChanged": [
  {
    "type": "SetValue",
    "property": "page",
    "value": "${event.source.value}"
  },
  {
    "when": "${pagerItems[page].pageDidAppear}",
    "type": "Sequential",
    "sequencer": "auto",
    "commands": [
      "${pagerItems[page].pageDidAppear}"
    ]
  }
]
```

## Why do I need onMount and onPageChanged?

The onPageChanged handler is only executed when the pager changes the page, but since our first page is already visible the onMount handler must be used.

## How do I know that the pager changes the page?

We will use a user-defined command to change the page in our pager. 

```
"NextPage": {
  "commands": [
    {
      "when": "${pagerItems[page].pageWillDisappear}",
      "type": "Sequential",
      "sequencer": "auto",
      "commands": [
        "${pagerItems[page].pageWillDisappear}"
      ]
    },
    {
      "when": "${page < pages}",
      "type": "SetPage",
      "sequencer": "auto",
      "position": "relative",
      "value": 1
    },
    {
      "when": "${page == pages - 1}",
      "type": "SetPage",
      "sequencer": "auto",
      "position": "absolute",
      "value": 0
    }
  ]
}
```

With our "NextPage" command we can check if commands are defined for the "pageWillDisappear" lifecycle event and execute them accordingly before we jump to the next page in our pager. Since we are using a pager with "navigation: none" set, we have to check if we have already reached the end and jump to the first page.

That's the whole magic behind the lifecycle events, we can define an object for each page in the data sources and specify which commands should be executed when entering or leaving a page. 

```
"datasources": {
  "currentView": {
    "type": "object",
    "properties": {
      "pagerItems": [
        {
          "content": {
            ...
          },
          "pageWillDisappear": [
            ...
          ],
          "pageDidAppear": [{
            "type": "Idle",
            "delay": 3000
          }, {
            "type": "NextPage"
          }]
        }
      ]
    }
  }
}
``` 

For each page, our user-defined command "NextPage" must be called in the pageDidAppear event, otherwise the page will never be changed.

## Why can I use variables that are not known in the custom command?

The command is executed in the scope of the component and therefore has access to the variables we have previously bound to the component.