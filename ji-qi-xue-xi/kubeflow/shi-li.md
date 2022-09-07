# 示例

### 示例01：通过 x 值预测 y 值

```python
import tensorflow as tf
import numpy as np

## 创建模型
# units = 1 表示单一神经元
# input_shape = [1] 表示输入值只有一个
model = tf.keras.Sequential([tf.keras.layers.Dense(units=1, input_shape=[1])])

## 为模型设置优化函数与损失函数
# 在训练时首先以损失函数来计算衡量结果的准确性
# 再以优化函数再次推算结果，两者结合提升结果的准确性
model.compile(optimizer="sgd",loss="mean_squared_error")

## 训练数据
# xs 表示用于训练的参考输入值
xs = np.array([-1.0,0.0,1.0,2.0,3.0,4.0])
# ys 表示用于训练的参考输出值
ys = np.array([-3.0,-1.0,1.0,3.0,5.0,7.0])

## 对模型进行训练，模拟 500 次，根据 xs 推演 ys 值
model.fit(xs,ys,epochs=500)

## 打印模型的预测结果，实际结果趋近于 y=2x-1，即 19
print(model.predict([10]))
```
