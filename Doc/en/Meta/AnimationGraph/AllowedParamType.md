# AllowedParamType

- **Usage location:** UFUNCTION
- **Engine module:** AnimationGraph
- **Metadata type:** string = "abc"

```cpp
// Sets a parameter's value in the supplied scope.
// @param    Scope    Scopes corresponding to an existing scope in a schedule, or "None". Passing "None" will apply the parameter to the whole schedule.
// @param    Ordering Where to apply the parameter in relation to the supplied scope. Ignored for scope "None".
// @param    Name     The name of the parameter to apply
// @param    Value    The value to set the parameter to
UFUNCTION(BlueprintCallable, Category = "AnimNext", CustomThunk, meta = (CustomStructureParam = Value, UnsafeDuringActorConstruction))
void SetParameterInScope(UPARAM(meta = (CustomWidget = "ParamName", AllowedParamType = "FAnimNextScope")) FName Scope, EAnimNextParameterScopeOrdering Ordering, UPARAM(meta = (CustomWidget = "ParamName")) FName Name, int32 Value);
```

Checked and found it is used exclusively in AnimNext.