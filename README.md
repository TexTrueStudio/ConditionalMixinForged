# ConditionalMixinForged

A forge library mod for using annotation to conditionally apply your mixins.

It is available at [jitpack](https://jitpack.io/#TexTrueStudio/ConditionalMixinForged)

## Example Usages

Import ConditionalMixinForged

```groovy
repositories {
    maven { url 'https://jitpack.io' }
}

dependencies {
    modImplementation 'com.github.TexTrueStudio:ConditionalMixinForged:forge~dev-SNAPSHOT'

    // suggested, to bundle it into your mod jar
    include "com.github.TexTrueStudio:ConditionalMixinForged:forge~dev-SNAPSHOT"
}
```

Inherit `RestrictiveMixinConfigPlugin` to create your mixin config plugin class

The `RestrictiveMixinConfigPlugin` will disable those mixins that don't satisfy with the annotated restriction in its `shouldApplyMixin` method

```java
public class MyMixinConfigPlugin extends RestrictiveMixinConfigPlugin
{
    // ...
}
```

Specify the mixin config plugin class in your mixin meta json:

```yaml
"plugin": "my.mod.MyMixinConfigPlugin",
```

Then you can annotate your mixins like:

```java
@Restriction(
        require = {
                @Condition("some_mod"),
                @Condition(value = "another_mod", versionPredicates = "2.0.x"),
                @Condition(value = "random_mod", versionPredicates = {">=1.0.1 <1.2", ">=2.0.0"}),
        }
)
@Mixin(SomeClass.class)
public abstract class SomeClassMixin
{
    // ...
}
```

or

```java
@Restriction(
        require = @Condition(type = Condition.Type.MIXIN, value = "my.mod.mixin.ImportantMixin"),
        conflict = @Condition("bad_mod")
)
@Mixin(AnotherClass.class)
public abstract class AnotherClassMixin
{
    // ...
}
```

You can also define your own tester class for more complicated condition testing

```java
@Restriction(
        require = @Condition(type = Condition.Type.TESTER, tester = MyConditionTester.class)
)
@Mixin(RandomClass.class)
public abstract class RandomClassMixin
{
    // ...
}

public class MyConditionTester implements ConditionTester
{
	@Override
	public boolean isSatisfied(String mixinClassName)
	{
		// More complicated checks go here
		return mixinClassName.length() % 2 == 0;
	}
}
```

## Notes

If you are upgrading conditional-mixin from older version, make sure to check if you have overwritten some methods in your mixin plugin class, since class `RestrictiveMixinConfigPlugin` might implement more methods of `IMixinConfigPlugin` in newer versions. e.g. `RestrictiveMixinConfigPlugin#preApply` and `RestrictiveMixinConfigPlugin#postApply` are added in `v0.2.0` for being able to automatically remove the `@Restriction` in the merged target class
