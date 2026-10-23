# Healthcheck AI for Laravel

![Healthcheck AI checks](art/banner.png)

A [Spatie Laravel Health](https://spatie.be/docs/laravel-health) add-on for
checking Laravel AI prompt chains, embedding chains, and agent registries.

[![Tests](https://github.com/eloquage/healthcheck-ai/actions/workflows/run-tests.yml/badge.svg)](https://github.com/eloquage/healthcheck-ai/actions/workflows/run-tests.yml)
[![Latest Version](https://img.shields.io/packagist/v/illuma-law/healthcheck-ai)](https://packagist.org/packages/illuma-law/healthcheck-ai)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Requires PHP 8.3+, Laravel 11–13, Laravel AI SDK 0.6+, and Spatie Laravel
Health 1.39+.

## Install

```bash
composer require illuma-law/healthcheck-ai
php artisan vendor:publish --tag=healthcheck-ai-config
```

## Register checks

Add the checks to your health registration. Configure the same provider and
model names used by your application. These checks make provider requests, so
enable caching and use a small, low-cost canary.

```php
use IllumaLaw\HealthCheckAi\AiPromptChainHealthCheck;
use IllumaLaw\HealthCheckAi\AiEmbeddingChainHealthCheck;
use Spatie\Health\Facades\Health;

Health::checks([
    AiPromptChainHealthCheck::new()
        ->resolveChainUsing(fn () => [
            ['provider' => 'openai', 'model' => 'gpt-4o-mini'],
        ]),

    AiEmbeddingChainHealthCheck::new()
        ->dimensions(1536)
        ->resolveChainUsing(fn () => [
            ['provider' => 'openai', 'model' => 'text-embedding-3-small'],
        ]),
]);
```

When a primary model fails but a fallback works, the chain check reports a
warning. The embedding check also warns when a returned vector has the wrong
dimension. The agent registry check validates configured agent classes and
credential availability:

```php
use IllumaLaw\HealthCheckAi\AiAgentRegistryCheck;

AiAgentRegistryCheck::new()
    ->resolveAgentsUsing(fn () => [App\Agents\SupportAgent::class])
    ->hasCredentialsUsing(fn (string $provider) => filled(config("ai.providers.{$provider}.api_key")));
```

Published settings in `config/healthcheck-ai.php` control cache TTLs, the
embedding dimension and canary text, and prompt timeout and canary text.

## Test

```bash
composer test
composer analyse
vendor/bin/pint --test
```

MIT licensed. See [LICENSE](LICENSE).
