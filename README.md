# authenticate-oauth-cli

CLI interface for authorizing OAuth2 applications via [authenticate-oauth][].

[authenticate-oauth]: https://hackage.haskell.org/package/authenticate-oauth

Your user will be presented with a URL to visit to authorize your application
and receive a PIN. Upon entering the correct PIN, your program receives control
again with a valid [`Credential`][].

[`Credential`]: https://hackage.haskell.org/package/authenticate-oauth/docs/Web-Authenticate-OAuth.html#t:Credential

The `Credential` is cached on the file system (following [XDG][]), meaning
subsequent calls to `authorizeApp` will not prompt, instead returning the cached
`Credential` immediately.

[xdg]: http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html

## Installation

```
git clone https://github.com/pbrisbin/authenticate-oauth-cli
cd your-project
cabal sandbox add-source path/to/authenticate-oauth-cli
```

## Usage

```haskell
{-# LANGUAGE OverloadedStrings #-}
module Example where

import Control.Monad.IO.Class (liftIO)
import Network.HTTP.Conduit (httpLbs, parseUrl, withManager)
import Web.Authenticate.OAuth (OAuth(..), newOAuth, signOAuth)
import Web.Authenticate.OAuth.CLI (authorizeApp)

main :: IO ()
main = withManager $ \m -> do
    cred <- authorizeApp "example" oauth m

    request <- signOAuth oauth cred =<< parseUrl "https://api.service.com/path"

    response <- httpLbs request m

    liftIO $ print response

oauth :: OAuth
oauth = newOAuth
    { oauthConsumerKey = "abc123"
    , oauthConsumerSecret = "abc123"
    , oauthRequestUri = "https://api.service.com/oauth/token"
    , oauthAccessTokenUri = "https://api.service.com/oauth/request_token"
    , oauthAuthorizeUri = "https://api.service.com/oauth/authorize_token"
    }
```

## Status

Not released.

Only known to work with Twitter. Known *not* to work with GitHub because their
API requires a User Agent which authenticate-oauth does not send by default,
apparently.

Cached `Credential`s expiring or otherwise becoming invalid is not handled
gracefully (or at all, really). You'll have to check for such a scenario
yourself, find and remove any cached file, and try again.
