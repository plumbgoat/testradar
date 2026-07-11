# Test framework detection & conventions

Detect the framework from evidence in the repo, never assume. Match the
project's existing style over any generic ideal.

## Detection signals per language

| Language | Look for (manifest) | Common frameworks | Test file convention |
|---|---|---|---|
| JavaScript/TypeScript | `package.json` devDeps | Jest, Vitest, Mocha+Chai, node:test | `*.test.js`/`*.spec.ts`, or `__tests__/` |
| Python | `pyproject.toml`, `requirements*.txt`, `setup.py`, `tox.ini` | pytest (dominant), unittest | `test_*.py` / `*_test.py`, `tests/` dir |
| Java | `pom.xml`, `build.gradle` | JUnit 5 (`org.junit.jupiter`), JUnit 4, TestNG; Mockito | `src/test/java/**/*Test.java` |
| Kotlin | gradle | JUnit5/Kotest; MockK | `src/test/kotlin/**` |
| Go | `go.mod` | stdlib `testing`, testify | `*_test.go` beside source |
| Ruby | `Gemfile` | RSpec, Minitest | `spec/**/*_spec.rb` / `test/**/*_test.rb` |
| C#/.NET | `*.csproj` | xUnit, NUnit, MSTest; Moq | `*Tests.cs`, separate test project |
| Rust | `Cargo.toml` | built-in `#[test]` | `#[cfg(test)] mod tests` in-file, or `tests/` |
| PHP | `composer.json` | PHPUnit, Pest | `tests/`, `*Test.php` |

**Decisive step:** open 1-2 existing test files and copy their actual
conventions — import style, test naming (`test_foo_returns_none_when_empty`
vs `shouldReturnNoneWhenEmpty` vs `foo returns none when empty`),
setup/teardown, fixture/factory usage, and mocking library. The existing
tests are the spec for the tests you write.

## Run commands (to verify new tests pass)

| Framework | Run all | Run one file/test |
|---|---|---|
| Jest | `npx jest` | `npx jest path/to/file.test.js -t "name"` |
| Vitest | `npx vitest run` | `npx vitest run path -t "name"` |
| pytest | `pytest -q` | `pytest path::test_name -q` |
| unittest | `python -m unittest` | `python -m unittest module.Class.test` |
| JUnit (Maven) | `mvn -q test` | `mvn -q -Dtest=ClassName#method test` |
| JUnit (Gradle) | `./gradlew test` | `./gradlew test --tests 'Class.method'` |
| Go | `go test ./...` | `go test -run TestName ./pkg` |
| RSpec | `bundle exec rspec` | `bundle exec rspec path:LINE` |
| .NET | `dotnet test` | `dotnet test --filter Name` |
| Cargo | `cargo test` | `cargo test test_name` |
| PHPUnit | `./vendor/bin/phpunit` | `./vendor/bin/phpunit --filter test` |

Run only the new/relevant tests when the suite is large and slow — target
the file or test name — but do a broader run if the new tests touch shared
fixtures.

## Coverage tools (optional signal, NOT the goal)

- JS: `jest --coverage` / `vitest run --coverage`
- Python: `pytest --cov`
- Java: JaCoCo (`mvn test` with the plugin; report in `target/site/jacoco`)
- Go: `go test -cover ./...`
- Ruby: SimpleCov
- .NET: `dotnet test --collect:"XPlat Code Coverage"`

Use coverage to find *which lines/branches* aren't exercised, then judge
whether they're worth testing. Never treat the percentage as the target —
covered-but-unasserted code is still an untested behavior.

## What is worth a test (judgment guide)

Worth testing: business logic, branching, error handling, boundary/edge
conditions, parsing/validation, state transitions, anything with a failure
mode or a "this must never break" invariant.

Usually not worth a dedicated test: trivial getters/setters, thin
delegation/pass-through, framework glue with no logic, generated code,
pure constants/config. Testing these to raise coverage % adds maintenance
cost with no safety gain.

## Behavior-not-implementation rule

A good test asserts what the unit *does* as seen from outside (return
value, thrown error, side effect, output), so it keeps passing when the
internals are refactored. A test that asserts private internals or exact
call sequences breaks on harmless refactors and trains people to ignore
failures — avoid it unless the interaction itself is the contract (e.g.
"must call the payment gateway exactly once").
