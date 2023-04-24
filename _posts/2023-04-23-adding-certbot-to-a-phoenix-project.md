---
layout: post
title: adding certbot to a phoenix project
---

the [phoenix](https://www.phoenixframework.org/) framework is the de-facto web server for the [elixir](elixir-lang.org/) ecosystem. it's production ready and doesn't need any sort of ingress like nginx in front of it. there's a section in the docs about how to [set up ssl](https://hexdocs.pm/phoenix/using_ssl.html) which does a great job, but you need to get the ssl certs first. which, most of the time, means you need [certbot](https://certbot.eff.org/)!

since you've skipped setting up `nginx` and `apache`, `certbot` can't auto-configure itself. you'll need to use the [`--webroot` option](https://eff-certbot.readthedocs.io/en/stable/using.html#webroot) instead, which means you'll need to configure phoenix to serve the challenges that `certbot` creates.

I found and tried [SiteEncrypt](https://github.com/sasa1977/site_encrypt) but it errored in some `GenServer` and I wasn't up to the task of digging into the stack trace. it turns out it's only a couple of lines of code anyways!

my first instinct was to use `Plug.Static`, since all I needed to do was serve the files in the `.well-known/` directory that `certbot` creates. this fell flat because I wanted to configure the `WELL_KNOW_PATH` at runtime, but `plug Plug.Static` seems to all happen at compile time. so have to add the route myself!

1. add a route for certbot

    ```
    --- a/lib/qian_bei_web/router.ex
    +++ b/lib/qian_bei_web/router.ex
    +
    +  # Certbot
    +  scope "/", QianBeiWeb do
    +    pipe_through :browser
    +
    +    get "/.well-known/*path", CertbotController, :challenge
    +  end
    ```

1. add the the `CerbotController`

    ```
    --- /dev/null
    +++ b/lib/qian_bei_web/controllers/certbot_controller.ex
    defmodule QianBeiWeb.CertbotController do
      @moduledoc "serves files at /.well-known/ for `certbot` challenges"
      use QianBeiWeb, :controller

      @spec challenge(Plug.Conn.t(), map()) :: Plug.Conn.t()
      def challenge(conn, %{"path" => path}) do
        with well_known_dir when not is_nil(well_known_dir) <-
               Application.fetch_env!(:qian_bei, :certbot)[:well_known_dir],
             path = Enum.join(path, "/"),
             file = Path.join(well_known_dir, path),
             true <- File.exists?(file) do
          conn
          |> Plug.Conn.send_file(:ok, file)
        else
          _ -> conn |> put_view(QianBeiWeb.ErrorView) |> render(:"404") |> halt()
        end
      end
    end
    ```

1. deploy your phoenix webapp, making sure to set `config :qian_bei, :certbot, well_known_dir: System.fetch_env!("WEBROOT_PATH")` (same as below)
1. get the certificates `sudo certbot -d "${DOMAIN:qianbei.com}" -v certonly --webroot --webroot-path="${WEBROOT_PATH:/var/www/qianbei}"`
1. configure your `Endpoint` to use the new certs

    ```
    --- a/config/runtime.exs
    +++ b/config/runtime.exs
    + config :qian_bei, QianBeiWeb.Endpoint,
    +   https: [
    +     certfile: System.fetch_env!("WEB_SSL_CERT_PATH") || "/etc/letsencrypt/live/qianbei.com/fullchain.pem",
    +     keyfile: System.fetch_env!("WEB_SSL_KEY_PATH") || "/etc/letsencrypt/live/qianbei.com/privkey.pem"
    +   ]
    ```

1. recompile/redoply/restart the server (note the server will have to be run under the `root` account to access `/et/letsencrypt`)
1. visit your website and make sure it's working!
1. set up autorenew `sudo certbot renew --dry-run` (this will add the autorenew to `/etc/crontab` or `/etc/cron.*/*` or `systemctl list-timers`)
