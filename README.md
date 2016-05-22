
## Installation

Oauth2 Server for Phoenix Framework

If [available in Hex](https://hex.pm/docs/publish), the package can be installed as:

  1. Add oauth2_server to your list of dependencies in `mix.exs`:

        def deps do
          [{:oauth2_server, "~> 0.1.1"}]
        end

  2. Ensure oauth2_server is started before your application:

        def application do
          [applications: [:oauth2_server]]
        end

## Req

NOTE : Postgres & MongoDB are not yet supported

You must have a table named `users` with the following fields:
  
  1. `id` bigint(20)
  2. `email` string
  3. `password` string

Use [comeonin](https://github.com/elixircnx/comeonin/) for password hashing

## Setup

1. Add these lines on your config.exs
    
    ```elixir
    config :oauth2_server, Oauth2Server.Repo,
      adapter: Ecto.Adapters.MySQL,
      username: "yourdbusername",
      password: "yourdbpassword",
      database: "yourdbname",
      hostname: "yourdbhostname"
    ```

    ```elixir
    config :oauth2_server, Oauth2Server.Settings, 
      access_token_expiration: 3600,
      refresh_token_expiration: 3600
    ```

2. Sample setup for endpoints that needs an access_token

    ```elixir
    pipeline :secured_api do
      plug :fetch_session
      plug :accepts, ["json"]

      plug Oauth2Server.Secured
    end
    ```

    ```elixir
    scope "/api", Phoenixtrial do
      pipe_through :api

      scope "/v1", v1, as: :v1 do
        post "/login", UserApiController, :login

        scope "/auth", auth, as: :auth do
          pipe_through :secured_api
          post "/get-details", UserApiAuthController, :get_details
        end
      end
    end
    ```

## Usage

```elixir
  $ mix ecto.migrate
  $ mix deps.get
  $ mix deps.compile
  $ mix compile
```

To create oauth tables execute the command :

```elixir
  $ mix oauth2_server.init
```

To create an Oauth client execute :

```elixir
  $ mix oauth2_server.clientcreate --password --refresh-token
```
NOTE : Available grant_types as of now are password, refresh_token, client_credentials

```elixir
  $ mix oauth2_server.clientcreate --password --refresh-token --client-credentials
```

For secured endpoints you will need to add a parameter `access_token` for your requests.
You can fetch the user id of the token owner via : get_session(conn, :oauth2_server_user_id)

## License

The Oauth2Server is released under the MIT license. See the LICENSE file.
