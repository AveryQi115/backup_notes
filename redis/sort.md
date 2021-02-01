## sort指令  
  
sort是redis系统中一个相当强大的功能。  
  
通过sort指令可以对列表，集合，有序集合类型进行排序。  
  
`SORT key` sort的基础使用，其中，如果对有序集合类型进行排序会按照元素自身的值进行排序，而不是分数  
  
`SORT key BY item:*->referenced [DESC]` BY代表参考键，item通常为待排序元素（对于散列类型有不一样的用法）  
  
使用参考键后排序时会根据参考键进行排序  
  
`SORT key BY item:*->referenced [DESC] GET item:*->return`  
  
使用get参数排序后返回return变量，一个sort指令中可以有多个get返回  
  
get # 返回元素本身
