# Embedded Go Authentication Framework

## Vision

A Go-first embedded authentication framework for applications.

Unlike Keycloak, Auth0, or SuperTokens, this project is **not** a
standalone authentication server or identity platform. The goal is to
provide a production-ready authentication engine that applications embed
directly into their own process.

The framework owns authentication logic while the application owns the
server.

## Non-goals

-   Not a hosted authentication product.
-   Not a replacement for enterprise IAM platforms such as Keycloak.
-   Not intended to act as a shared identity provider for multiple
    independent applications.
-   OAuth2/OIDC provider support is not a primary goal.

## High-Level Architecture

``` text
Application
    |
    +-- HTTP (net/http, Gin, Echo, Fiber, ...)
    |        |
    |        +-- Registers auth routes
    |
    +-- auth-core
              |
              +-- Storage Adapter
              +-- Cache Adapter
              +-- Key Store Adapter
              +-- Typed Event Hooks
              +-- Policies
              +-- Logger
```

## Core Responsibilities

-   User management
-   Email/password authentication
-   Magic link authentication
-   MFA (future: TOTP/WebAuthn)
-   Session management
-   JWT issuance and validation
-   Refresh tokens
-   Password reset
-   Email verification
-   Administration services
-   Automatic database migrations

## Transport Philosophy

The framework does **not** own an HTTP server.

Instead it exposes route registration for frameworks such as:

-   net/http
-   Gin
-   Echo
-   Fiber
-   (future) gRPC / Connect

Applications register the endpoints into their existing server.

## Storage

Storage is pluggable.

Initial focus is relational databases only.

Planned implementations:

-   PostgreSQL
-   MySQL
-   SQLite

Users may implement their own storage adapter.

The framework owns:

-   schema
-   migrations
-   indexes
-   constraints

Direct database modification is unsupported, although users ultimately
control their own database.

## Cache

Adapter-based.

Default:

-   In-memory

Planned:

-   Redis

Cache is optional infrastructure.

## Key Store

Signing keys are abstracted behind a KeyStore interface.

Possible implementations:

-   Database
-   AWS KMS
-   HashiCorp Vault
-   Custom implementations

Future features:

-   Automatic signing-key rotation
-   Multiple active keys
-   Sliding validation window
-   Configurable rotation policies

## Typed Event Hooks

The framework emits typed in-process events.

Examples:

-   UserCreated
-   UserDeleted
-   LoginSucceeded
-   LoginFailed
-   PasswordChanged
-   MFAEnabled
-   SessionCreated
-   SessionRevoked

Applications decide what to do:

-   publish to Kafka
-   send email
-   invoke Temporal
-   update CRM
-   metrics
-   cache invalidation

The framework itself has no dependency on a message broker.

## Policies

Authentication behavior should be configurable through policies.

Examples:

-   Password policy
-   Username policy
-   Email policy
-   MFA policy
-   Session policy

## Logging

The framework should integrate with application logging.

Goals:

-   configurable log level
-   logger abstraction (ideally compatible with slog)

Never log:

-   passwords
-   hashes
-   JWTs
-   refresh tokens
-   secrets
-   private keys

## Administration

Core services should expose administration operations such as:

-   List users
-   Disable users
-   Force password reset
-   Revoke sessions

HTTP endpoints are adapters over these services.

## Current State vs Audit History

The framework stores operational state:

-   last login
-   password changed at
-   active sessions
-   failed login count
-   account lock status

Full audit history is intentionally postponed.

Applications requiring audit trails can consume typed events and persist
them independently.

## Guiding Principles

-   Embedded, not hosted.
-   Core first; transports are adapters.
-   Composition over monolith.
-   Strong defaults with extensibility.
-   SQL-first.
-   Typed APIs over generic maps.
-   Infrastructure behind small interfaces.
-   Avoid unnecessary abstraction.
