# Getting started

Creating an action for a GPT takes 3 steps:

1. Build an API
2. Document the API in the OpenAPI YAML or JSON format
3. Expose the Schema to your GPT in the ChatGPT UI

The focus of the rest of this section will be creating a TODO list GPT by defining the custom action for the GPT.

If you want to kickstart the process of creating your GPT schema, you can use the [experimental ActionsGPT](https://chat.openai.com/g/g-TYEliDU6A-actionsgpt). Keep in mind the known limitation when building a GPT.

## Schema definition

Once you create the basics of a TODO GPT, the next step is to build the OpenAPI specification to document the API. The model in ChatGPT is only aware of your API structure as defined in the schema. If your API is extensive, you don't need to expose all its functionality to the model; you can choose only specific endpoints to include. For example, if you have a social media API, you might want to have the model access content from the site through a GET request but prevent the model from being able to comment on users posts in order to reduce the chance of spam.

The OpenAPI specification is the wrapper that sits on top of your API. A basic OpenAPI specification will look like the following:

```yaml
openapi: 3.0.1
info:
  title: TODO Action
  description: An action that allows the user to create and manage a TODO list using a GPT.
  version: 'v1'
servers:
  - url: https://example.com
paths:
  /todos:
    get:
       operationId: getTodos
      summary: Get the list of todos
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
components:
  schemas:
    getTodosResponse:
      type: object
      properties:
        todos:
          type: array
          items:
            type: string
          description: The list of todos.
```

We start by defining the specification version, the title, description, and version number. When a query is run in ChatGPT, it will look at the description that is defined in the info section to determine if the action is relevant for the user query. You can read more about prompting in the writing descriptions section.

Keep in mind the following limits in your OpenAPI specification, which are subject to change:

- 300 characters max for each API endpoint description/summary field in API specification
- 700 characters max for each API parameter description field in API specification

The OpenAPI specification follows the traditional OpenAPI format, you can learn more about OpenAPI formatting and how it works. There are also many tools that auto generate OpenAPI specifications based on your underlying API code.

### Hosted OpenAPI specification

With Actions, we host the OpenAPI specification for your API in order to track changes. You can import an existing OpenAPI specification or create a new one from scratch using the UI in the GPT creator.

### Consequential flag

In the OpenAPI specification, you can now set certain endpoints as "consequential" as shown below:

```yaml
paths:
  /todo:
    get:
      operationId: getTODOs
      description: Fetches items in a TODO list from the API.
      security: []
    post:
      operationId: updateTODOs
      description: Mutates the TODO list.
      x-openai-isConsequential: true
```

A good example of a consequential action is booking a hotel room and paying for it on behalf of a user.

- If the `x-openai-isConsequential` field is `true`, we treat the operation as "must always prompt the user for confirmation before running" and don't show an "always allow" button (both are features of GPTs designed to give builders and users more control over actions).
- If the `x-openai-isConsequential` field is `false`, we show the "always allow button".
- If the field isn't present, we default all GET operations to `false` and all other operations to `true`

### Multiple authentication schemas

When defining an action, you can mix a single authentication type (OAuth or API key) along with endpoints that do not require authentication.

You can learn more about action authentication on our actions authentication page.

## Testing an action

In the GPT editor, once you have added an action, a new section below the schema will appear called "Available actions", this is generated by parsing the schema. You can preview the name, method, and path the action lives at. There will also be a "Test" button displayed which allows you to try your actions. After you press "Test", in the preview section of the GPT editor you will be presented with a request to "Allow", "Always allow", or "Decline" to run the action. These are user confirmations designed to given end users more control over what an action does.

There are also various debugging information made available inside the preview mode which should help you understand any unintended behavior. If everything is working as expected, you can save or update your GPT in the top right corner.

## Writing descriptions

When a user makes a query that might trigger an action, the model looks through the descriptions of the endpoints in the schema. Just like with prompting other language models, you will want to test out multiple prompts and descriptions to see what works best.

The schema is a great place to provide the model with detailed information about your API, such as the available functions and their parameters. Besides using expressive, informative names for each field, the schema can also contain "description" fields for every attribute. You can use these fields to provide natural language descriptions that explain what each method does or what information a query field requires. The model will be able to see these, and they will guide it in using the API. If a field is restricted to only certain values, you can also provide an "enum" with descriptive category names.

The instructions for a GPT gives you the freedom to instruct the GPT on how to use your action generally. Overall, the language model behind ChatGPT is highly capable of understanding natural language and following instructions. Therefore, this is a good place to put in general instructions on what your action does and how the GPT should use it properly. Use natural language, preferably in a concise yet descriptive and objective tone. You can look at some of the examples to have an idea of what this should look like.

### Best practices

Here are some best practices to follow when writing your GPT instructions and descriptions in your schema, as well as when designing your API responses:

1. Your descriptions should not encourage the GPT to use the action when the user hasn't asked for your action's particular category of service.

    _Bad example_:

    > Whenever the user mentions any type of task, ask if they would like to use the TODO action to add something to their todo list.

    _Good example_:

    > The TODO list can add, remove and view the user's TODOs.

2. Your descriptions should not prescribe specific triggers for the GPT to use the action. ChatGPT is designed to use your action automatically when appropriate.

    _Bad example_:

    > When the user mentions a task, respond with "Would you like me to add this to your TODO list? Say 'yes' to continue."

    _Good example_:

    > \[no instructions needed for this\]

3. Action responses from an API should return raw data instead of natural language responses unless it's necessary. The GPT will provide its own natural language response using the returned data.

    _Bad example_:

    > I was able to find your todo list! You have 2 todos: get groceries and walk the dog. I can add more todos if you'd like!

    _Good example_:

    > { "todos": \[ "get groceries", "walk the dog" \] }

## Limitations

There are a few limitations to be aware of when building with actions:

- Custom headers are not supported
- With the exception of Google, Microsoft and Adobe OAuth domains, all domains used in an OAuth flow must be the same as the domain used for the primary endpoints
- Request and response payloads must be less than 100,000 characters each
- Requests timeout after 45 seconds
- Requests and responses can only contain text (no images or video)

If you have questions or run into additional limitations, you can join the discussion on the OpenAI developer forum.