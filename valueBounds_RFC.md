# [RFC] Update valueBoundsOpInterface to compute bounds of dynamic stride/offset sizes

### Background

The `valueBoundsOpInterface` is a great design in mlir, widely used for analysis and transformation passes. However, its inability to compute dynamic stride or offset sizes presents a limitation. 

Given the prominence of `stride` and `offset` in the `mlir::MemRef` dialect, addressing this deficiency is essential. We propose extending the `valueBoundsOpInterface` to incorporate dynamic stride and offset calculations.

### Extend Interface Methods
```c++
void populateBoundsForOffsetValue(Value value, ValueBoundsConstraintSet &cstr)

void populateBoundsForStridedValueDim(Value value, int64_t dim, ValueBoundsConstraintSet &cstr)
```

The `ValueDim` and `ValueDimList` within `ValueBoundsConstraintSet` are insufficient for capturing the necessary information about `IndexValue`, `ShapedValueDim`, `StridedValueDim`, and `OffsetValue`. To address this, we propose introducing a `PopulateMode` enum and a `ColumnNode` class.  `PopulateMode` specifies the target computation (e.g., IndexValue, ShapedValue, StridedValue, or OffsetValue). The `ColumnNode` containing `value`, `dim`, and `PopulateMode`, can provide comprehensive information required for dynamic size calculations.

The demo code likes belows: 

```c++
enum PopulateMode {
  INDEX = 0,
  SHAPE,
  STRIDE,
  OFFSET,
};

struct ColumnNode {
  Value value;
  int64_t dim;
  PopulateMode pMode;
};

using ColumnNodeList = SmallVector<ColumnNode>;

class Variable {
// ...
// other code
// ...
private:
  friend class ValueBoundsConstraintSet;
  AffineMap map;
  ColumnNodeList mapOperands;
};
```
