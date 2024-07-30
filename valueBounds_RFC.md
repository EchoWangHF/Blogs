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

The demo code is as follows: 

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

Some other functions within `ValueBoundsConstraintSet`, including `getExpr`, `getPos`, and `processWorklist`, require corresponding adjustments due to the integration of `ColumnNode` and `PopulateMode`. However, these modifications are relatively straightforward and can be implemented efficiently.

### Example: memref::CastOp
```c++
struct CastOpInterface
    : public ValueBoundsOpInterface::ExternalModel<CastOpInterface, CastOp> {
  void populateBoundsForShapedValueDim(Operation *op, Value value, int64_t dim,
                                       ValueBoundsConstraintSet &cstr) const {
    auto castOp = cast<CastOp>(op);
    assert(value == castOp.getResult() && "invalid value");

    if (llvm::isa<MemRefType>(castOp.getResult().getType()) &&
        llvm::isa<MemRefType>(castOp.getSource().getType())) {
      cstr.bound(value)[dim] == cstr.getExpr(castOp.getSource(), dim);
      cstr.bound({value, dim, PopulateMode::SHAPE}) ==
          cstr.getExpr(castOp.getSource(), dim, PopulateMode::SHAPE);
    }
  }

  void populateBoundsForOffsetValue(Operation *op, Value value,
                                    ValueBoundsConstraintSet &cstr) const {
    auto castOp = cast<CastOp>(op);
    assert(value == castOp.getResult() && "invalid value");

    cstr.bound(value, std::nullopt, PopulateMode::OFFSET) ==
        cstr.getExpr(castOp.getSource(), std::nullopt, PopulateMode::OFFSET);
  }

  void populateBoundsForStridedValueDim(Operation *op, Value value, int64_t dim,
                                        ValueBoundsConstraintSet &cstr) const {
    auto castOp = cast<CastOp>(op);
    assert(value == castOp.getResult() && "invalid value");

    if (llvm::isa<MemRefType>(castOp.getResult().getType()) &&
        llvm::isa<MemRefType>(castOp.getSource().getType())) {
      cstr.bound(value, dim, PopulateMode::STRIDE) ==
          cstr.getExpr(castOp.getSource(), dim, PopulateMode::STRIDE);
    }
  }
};
```

### Example: memref::ReinterpretCastOpInterface
```c++
struct ReinterpretCastOpInterface
    : public ValueBoundsOpInterface::ExternalModel<ReinterpretCastOpInterface,
                                                   memref::ReinterpretCastOp> {
  void populateBoundsForOffsetValue(Operation *op, Value value,
                                    ValueBoundsConstraintSet &cstr) const {
    auto reinterpretCastOp = cast<memref::ReinterpretCastOp>(op);
    assert(value == reinterpretCastOp.getResult() && "invalid value");

    OpFoldResult offset = reinterpretCastOp.getConstifiedMixedOffset();
    cstr.bound(value, std::nullopt, PopulateMode::OFFSET) == offset;
  }

  void populateBoundsForShapedValueDim(Operation *op, Value value, int64_t dim,
                                       ValueBoundsConstraintSet &cstr) const {
    auto reinterpretCastOp = cast<memref::ReinterpretCastOp>(op);
    assert(value == reinterpretCastOp.getResult() && "invalid value");

    OpFoldResult shape = reinterpretCastOp.getConstifiedMixedSizes()[dim];
    cstr.bound(value, dim, PopulateMode::SHAPE) == shape;
  }

  void populateBoundsForStridedValueDim(Operation *op, Value value, int64_t dim,
                                        ValueBoundsConstraintSet &cstr) const {
    auto reinterpretCastOp = cast<memref::ReinterpretCastOp>(op);
    assert(value == reinterpretCastOp.getResult() && "invalid value");

    OpFoldResult stride = reinterpretCastOp.getConstifiedMixedStrides()[dim];
    cstr.bound(value, dim, PopulateMode::STRIDE) == stride;
  }
};
```

### summary:
To effectively address the increasing need for dynamic stride and offset calculations in various `Dialect`, the `valueBoundsOpInterface` should be extended to support these computations. Introducing `PopulateMode` and `ColumnNode` provides a direct and efficient solution.

### Replay
Firstly, thank you very much for taking the time to reply.

Well, the most important application of `valueBoundsOpInterface` is in memref dimension folding pass in my wrok.

### Example about memref dimension folding pass
```
func.func @demo(%arg0: memref<2x4xf32, 101>, %arg1: memref<2x4xf32, 101>) {
  toy.copy(%arg1, %arg0) : (memref<2x4xf32, 101>, memref<2x4xf32, 101>) -> ()
  return
}

--------> after dimension folding pass

func.func @demo(%arg0: memref<2x4xf32, 101>, %arg1: memref<2x4xf32, 101>) {
  %reinterpret_cast = memref.reinterpret_cast %arg0 to offset: [0], sizes: [8], strides: [1] : memref<2x4xf32, 101> to memref<8xf32, 101>
  %reinterpret_cast_0 = memref.reinterpret_cast %arg1 to offset: [0], sizes: [8], strides: [1] : memref<2x4xf32, 101> to memref<8xf32, 101>
  toy.copy(%reinterpret_cast_0, %reinterpret_cast) : (memref<8xf32, 101>, memref<8xf32, 101>) -> ()
  return
}
```
After dimension folding pass， the two-dimensional memref is folded into one dimension, making it more convenient for subsequent pass optimization.

In dimension folding pass, the key task is to determine whether the memref is continuous or identity. 

For example：

```
%arg0: memref<2x4xf32, 101>

%dim0 = 2

%dim1 = 4

%stride0 = 4

%stride1 = 1

if %stride0 == %dim1 * %stride1
  isContinuous = true
else 
  isContinuous = false

If the data in dim0 and dim1 is continuous, the dim0 and dim1 can be folded into one dimension.
```
While the provided example demonstrates a simple scenario with static shape and stride sizes, real-world applications often involve dynamic values. To effectively handle these cases, it is essential to reify dynamic stride and shape sizes using the `valueBoundsOpInterface`. This capability is crucial for accurate analysis and optimization.



