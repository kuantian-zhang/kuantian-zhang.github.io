# lance 学习笔记

lance 数据集过大，可以 lance.write_data 也是支持 pyarrow.RecordBatch Iterator，只需要提供 pyarrow.schema 即可

```python
import lance
import pyarrow as pa
from collections.abc import Iterator

def producer()-> Iterator[pa.RecordBatch]:
    yield pa.RecordBatch.from_pylist([{"column_1": "val_1_1", "column_2": 1}])
    yield pa.RecordBatch.from_pylist([{"column_1": "val_2_1", "column_2": 2}])
schema = pa.schema([
    ("column_1", pa.string()),
    ("column_2", pa.int32()),
])

ds = lance.write_dataset(producer(),
                         "./batch_column.lance",
                         schema=schema, mode="overwrite")
print(ds.count_rows())  # Output: 2
```

lance.write_data() 支持 ```pyarrow.Table```,```pandas.DataFrame```,```pyarrow.dataset.DataSet``` 和 ```Iterator[pyarrow.RecordBatch]``` 四种类型的数据

























