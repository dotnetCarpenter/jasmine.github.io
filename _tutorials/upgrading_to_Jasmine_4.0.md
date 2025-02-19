---
layout: faq # FAQ styling for lists and code blocks is right for this page
omit_from_index: true
---

<h1>Upgrading to Jasmine 4.0</h1>

<div class="warning">
	This is a rough draft that describes an unreleased version of Jasmine.
	Things will change between the time you read this and the final release.
</div>

<h2>Overview</h2>

**tl;dr**: Update to 3.99 and fix any deprecation warnings before updating to 4.0.

We think that Jasmine 4.0 will be an easy upgrade for most users. However,
it does contain a number of breaking changes and some people will need to
make changes to their code in order to upgrade. Following the steps in this
guide will give you the greatest chance of a smooth upgrade experience.

Breaking changes in Jasmine 4.0 include changes to how Jasmine is configured,
changes to custom matchers and asymmetric equality testers, better detection
of common errors, and changes to the system requirements. You can find a
complete list of breaking changes in the 
[release notes](https://github.com/jasmine/jasmine/blob/main/release_notes/4.0.0.md).

<h2>System requirements</h2>

The following previously-supported environments are no longer supported:
[ TODO: Update shortly before release. ]

* Node <12.17
* Internet Explorer
* Safari 8-12
* PhantomJS
* Python
* TODO: Ruby & Rails status & supported versions
* Bower

<h2>Using Jasmine 3.99 to detect compatibility problems</h2>

Jasmine 3.99 issues deprecation warnings for most code that uses APIs that are
removed or changed in incompatible ways in 4.0. We recommend upgrading to 3.99
and fixing all deprecation warnings before upgrading to 4.0. You should upgrade
the Jasmine packages that you depend on directly, as usual. If you upgrade the
`jasmine` NPM package, the `jasmine` Rubygem, or the `jasmine` Python package to
3.99, you'll also get version 3.99 of jasmine-core.

TODO: instructions for jasmine-browser-runner users

<h2>Notes for Ruby users</h2>

TODO

<h2>Notes for Python users</h2>

3.99 is the final version of Jasmine for Python. We recommend migrating to one
of the following alternatives:

* The [jasmine-browser-runner](https://github.com/jasmine/jasmine-browser)
npm package to run specs in browsers, including headless Chrome and
Saucelabs. This is the most direct replacement for the `jasmine server`
and `jasmine ci` commands provided by the `jasmine` Python package.
* The [jasmine](https://github.com/jasmine/jasmine-npm) npm package to run
specs under Node.js.
* The [standalone distribution](https://github.com/jasmine/jasmine#installation)
to run specs in browsers with no additional tools.
* The [jasmine-core](https://github.com/jasmine/jasmine) npm package if all
you need is the Jasmine assets. This is the direct equivalent of the
`jasmine-core` Python package.

<h2>Exit code changes</h2>

Prior to version 4, jasmine-npm exited with a status of 1 in almost all 
scenarios other than complete success. Version 4 uses different exit codes for
different types of failures. You don't need to change anything unless your build
or CI system specifically checks for an exit code of 1 (this is very unusual).

<h2>Tips for resolving specific deprecation warnings</h2>

<h3 id="matchers-cet">Deprecations related to custom equality testers in matchers</h3>

* "The matcher factory for [name] accepts custom equality testers, but this
  parameter will no longer be passed in a future release"
* "Passing custom equality testers to MatchersUtil#contains is deprecated."
* "Passing custom equality testers to MatchersUtil#equals is deprecated."
* "Diff builder should be passed as the third argument to MatchersUtil#equals,
  not the fourth."

Prior to 3.6, [custom matchers]({{ site.github.url }}/tutorials/custom_matcher)
that wanted to support [custom equality testers]({{ site.github.url }}/tutorials/custom_equality)
had to accept an array of custom equality testers as the matcher factory's 
second argument and pass it to methods like `MatchersUtil#equals` and
`MatchersUtil#contains`. Since 3.6, the `MatchersUtil` instance passed to the
matcher factory is preconfigured with the current set of custom equality
testers, and matchers do not need to provide the. Beginning with Jasmine 4.0,
custom equality testers are neither passed to the matcher factory nor accepted
as parameters to `MatchersUtil` methods. To update your custom matchers to the
new style, simply remove the extra parameter from the matcher factory and stop
passing it to `MatchersUtil` methods.

Before:

```
jasmine.addMatchers({
  toContain42: function(matchersUtil, customEqualityTesters) {
    return {
      compare: function(actual, expected) {
        return {
          pass: matchersUtil.contains(actual, 42, customEqualityTesters)
        };
      }
    };
  }
});
```


After:

```
jasmine.addMatchers({
  toContain42: function(matchersUtil) {
    return {
      compare: function(actual, expected) {
        return {
          pass: matchersUtil.contains(actual, 42)
        };
      }
    };
  }
});
```

<h4>Note to library authors</h4>

The updated form above is only compatible with Jasmine 3.6 and newer. If you
want to maintain compatibility with older versions of Jasmine, you can avoid the
deprecation warnings by declaring the matcher factory to take a single argument
and then using the `arguments` object or
[rest parameter syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
to access the array of custom equality testers. Then check for a `deprecated`
property on the array, which will be present in Jasmine 3.6 and later, before
passing it along:

````
jasmine.addMatchers({
  toContain42: function(matchersUtil, ...extraArgs) {
    const customEqualityTesters = 
      extraArgs[0] && !extraArgs[0].deprecated ? extraArgs[0] : undefined;
    
    return {
      compare: function(actual, expected) {
        return {
          pass: matchersUtil.contains(actual, 42, customEqualityTesters)
        };
      }
    };
  }
});
````

<h3 id="asymmetricMatch-cet">"The second argument to asymmetricMatch is now a
MatchersUtil. Using it as an array of custom equality testers is deprecated and
will stop working in a future release."</h3>

Prior to 3.6, [custom asymmetric equality testers]({{ site.github.url }}/tutorials/custom_asymmetric_equality_testers)
that wanted to support [custom equality testers]({{ site.github.url }}/tutorials/custom_equality)
had to accept an array of custom equality testers as second argument to
`asymmetricMatch` and pass it to methods like `MatchersUtil#equals` and
`MatchersUtil#contains`. Since 3.6, the second argument is both an array of
custom equality testers and a `MatchersUtil` instance that's preconfigured with
them. To resolve the deprecation warning, use the provided `MatchersUtil`
directly:

Before:

```
function somethingContaining42() {
  return {
    asymmetricMatch: function(other, customEqualityTesters) {
      return jasmine.matchersUtil.contains(other, 42, customEqualityTesters);
    }
  };
}
```


After:

```
function somethingContaining42() {
  return {
    asymmetricMatch: function(other, matchersUtil) {
      return matchersUtil.contains(other, 42);
    }
  };
}
```

<h4>Note to library authors</h4>

The updated form above is only compatible with Jasmine 3.6 and newer. If you
want to maintain compatibility with older versions of Jasmine, you can avoid the
deprecation warnings by checking whether the second argument is a `MatchersUtil`
(e.g. by checking for an `equals` method) and falling back to the old form above
if it isn't a `MatchersUtil`.


<h3 id="static-utils">Deprecations related to matcher utilities that are no 
longer available globally</h3>

* "jasmine.matchersUtil is deprecated and will be removed in a future release.
  Use the instance passed to the matcher factory or the asymmetric equality 
  tester's `asymmetricMatch` method instead."
* "jasmine.pp is deprecated and will be removed in a future release. Use the 
  pp method of the matchersUtil passed to the matcher factory or the asymmetric
  equality tester's `asymmetricMatch` method instead."

Until Jasmine 4.0 there was a static `MatchersUtil` instance available as
`jasmine.matchersUtil` and a static pretty printer available as `jasmine.pp`. 
Because both of those now carry configuration that's specific to the current
spec, it no longer makes sense to have static instances. Instead of using 
`jasmine.matchersUtil`, access the current `MatchersUtil` in one of the 
following ways:

* [The first parameter to a matcher factory]({{ site.github.url}}/tutorials/custom_matcher)
* [The second parameter to an asymmetric equality tester's `asymmetricMatch` method]({{ site.github.url }}/api/edge/AsymmetricEqualityTester.html#asymmetricMatch)

Instead of using `jasmine.pp`, access a `matchersUtil` in either of the above
ways and then use its [pp method]({{ site.github.url }}/api/edge/MatchersUtil.html#pp).

* The `pp` method of [the first parameter to a matcher factory]({{ site.github.url}}/tutorials/custom_matcher)
* The first parameter to an object's (particularly an asymmetric equality 
  tester's) `jasmineToString` method.


<h4>Note to library authors</h4>

`matchersUtil` is provided to matcher factories in all versions since 2.0, and
to asymmetric equality testers in all versions since 2.6. `matchersUtil#pp` was
introduced in 3.6, which is also the first version that passed a pretty-printer
to `jasmineToString`. If you need to pretty-print in a way that's compatible with
Jasmine versions older than 3.6, you can check for a `pp` method on the 
`matchersUtil` instance or a parameter passed to `jasmineToString` and then 
fall back to `jasmine.pp` if it isn't there.


<h3>Deprecations related to mixing two forms of async</h3>


* "An asynchronous before/it/after function was defined with the async 
  keyword but also took a done callback. This is not supported and will stop 
  working in the future. Either remove the done callback (recommended) or remove
  the async keyword."
* "An asynchronous before/it/after function took a done callback but also 
  returned a promise. This is not supported and will stop working in the future.
  Either remove the done callback (recommended) or change the function to not 
  return a promise."

Jasmine never supported functions that combine multiple forms of async, and
they never had a consistent or well-defined behavior. Jasmine 4.0 will issue a
spec failure whenever it encounters such a function.
[The FAQ discusses the reason for this change and how to update your specs]({{ site.github.url}}/pages/faq.html#010-mixed-style).
