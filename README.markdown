# Cucumber for Haskell

> An effort to bring Cucumber-ish BDD to Haskell.

Want to help?  Join in at `#hspec` on freenode!

## Sketch

Let's start with a simple feature:

```feature
Feature: Beta reduction of lambda terms

  In order to be able to reduce lambda terms
  As a programmer
  I want a function that reduces lambda terms

  Scenario: Reducing a simple lambda term
    Given a lambda term "(λx.x)y"
    When I parse it
    And I reduce it
    And I pretty-print it
    Then the result should be "y"
```

This should result in the following _test term_:

```haskell
test = (show . reduce . parse) "(λx.x)y" `shouldBe` "y"
```

Or:

```haskell
test = flip shouldBe "y" (show (reduce (parse (id "(λx.x)y"))))
```

So how would we give step definitions for that?  Currently I think Template
Haskell is our best bet. (I do not really like it!  If you have any other
ideas, please let me know.)

```haskell
$(given "a lambda term \"([^\"]*)\"") = id

$(when "I parse it") = parse

$(when "I reduce it") = reduce

$(when "I pretty-print it") = show

$(then_ "the result should be \"([^\"]*)\"") = flip shouldBe
```

This would then expand to something like:

```haskell
given_0 :: String -> Maybe String
given_0 = case match input "a lambda term \"([^\"]*)\"" of
  [x] -> Just (id x)
  _   -> Nothing

when_0 :: String -> Maybe (String -> Term)
when_0 input
  | input == "I parse it" = Just parse
  | otherwise             = Nothing

when_1 :: String -> Maybe (Term -> Term)
when_1 input
  | input == "I reduce it" = Just reduce
  | otherwise              = Nothing

when_2 :: Show a => String -> Maybe (a -> String)
when_2 input
  | input == "I pretty-print it" = Just show
  | otherwise                    = Nothing

then_0 :: String -> Maybe Expectation
then_0 input = case match input "the result should be \"([^\"]*)\"" of
  [x] -> Just (flip shouldBe x)
  _   -> Nothing
```

In theory, constructing the test term from that is easy:

```haskell
import Data.Maybe
import Control.Applicative

test = fromJust $
      then_0  "the result should be \"y\""
  <*> when_2  "I pretty-print it"
  <*> when_1  "I reduce it"
  <*> when_0  "I parse it"
  <*> given_0 "a lambda term \"(λx.x)y\""
```

### What to do if multiple things are given?

The resulting values could than be applied to the term we get from the first
`When` step.

## Support

 * https://github.com/sakari/haskell-gherkin
 * https://github.com/cucumber/cucumber-tck
 * https://github.com/cucumber/cucumber-bootstrap

## Related

Other efforts to implement a Cucumber-ish BDD framework for Haskell:

 * https://github.com/sakari/haskell-cucumber (relies on state!)

 * https://github.com/confucius/ConcombreAuCurry (nothing here yet!)
