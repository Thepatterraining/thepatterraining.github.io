---
title: php实现归并排序算法
date: 2022-02-18 10:12:47
tags: ['php','算法','归并排序']
category: php
article: php实现归并排序算法
---

# php实现归并排序算法

归并排序算法的复杂度是O(nlogn)。

代码如下,完整代码在[github上面](https://github.com/Thepatterraining/design-pattern)，只需要clone下来执行`composer install`然后执行 `php artisan test:mergeSort` 就可以看到结果了

```php
    /**
     * 归并排序把数据逐步分解，然后对分解后的数据进行排序，最后合并到一起
     *
     * @return mixed
     */
    public function handle()
    {
        $this->a = [3,70,4,38,5,6,8,4,7,10,6,10,34,4];
        dump($this->a);
        $a = $this->mergeSort($this->a, 0, count($this->a));
        dd($a);
    }

    private function mergeSort($a, $lo, $hi) {
        if (($hi - $lo) < 2) return [$a[$lo]];
        $mi = ($lo + $hi) >> 1;
        //把中点左边的进行归并
        $b = $this->mergeSort($a, $lo, $mi);
        dump('$b:',$b);
        //把中点右边的进行归并
        $c = $this->mergeSort($a, $mi, $hi);
        dump('$c:',$c);
        //把所有数据进行排序
        return $this->merge($b, $c, $lo,$mi,$hi);
    }

    /**
     * 假设有一个数组$a分成了两个数组[3,4] [2,8]
     * 逐一比较，3and2，取出来2然后3and8取出来3然后4and8取出来4，最后取出来8
     *
     * @param [type] $lo
     * @param [type] $mi
     * @param [type] $hi
     * @return void
     */
    private function merge($b, $c, $lo, $mi, $hi) {
        $lb = $mi - $lo; //$b数组的边界
        $lc = $hi - $mi; //$c数组的边界
        $res = [];
        //$i表示合并后数组的下标 $ib是b数组的下标 $ic是c数组的下标 
        for($i = 0,$ib=0,$ic=0;$ib<$lb || $ic < $lc;){
            //ib 下标没有越界 && c的数组已经空了也就是$ic >= $lc || 比较两个数组首位的大小 如果b的首元素 < c的首元素，那么取出来b的首元素
            if ($ib < $lb && ( $ic >= $lc || $b[$ib] <= $c[$ic])) {
                $res[$i++] = $b[$ib++];
            }
            //k 下标没有越界 && b的数组已经空了也就是$ib >= $lb || 如果c的首元素 < b的首元素，那么取出来c的首元素 
            if ($ic < $lc && ($ib >= $lb || $b[$ib] > $c[$ic])) {
                $res[$i++] = $c[$ic++];
            }
        }
        return $res;
    }
```


### 归并排序原理

归并排序和快排刚好相反，是先将整个数组左右打散，然后在逐一合并进行排序，最终完成整个数组的排序，排序示意图如下：
!(mpdf)[../images/mergesort01.png]
首先将整个数组左右打散，变成单个元素，因为单个元素可以被认为是有序的。
对应代码
```php
if (($hi - $lo) < 2) return [$a[$lo]];
$mi = ($lo + $hi) >> 1;
//把中点左边的进行归并
$b = $this->mergeSort($a, $lo, $mi);
dump('$b:',$b);
//把中点右边的进行归并
$c = $this->mergeSort($a, $mi, $hi);
dump('$c:',$c);
```

接下来对左右两个有序数组进行排序，假设有一个数组$a分成了两个数组[3,4] [2,8]，逐一比较，3and2，取出来2然后3and8取出来3然后4and8取出来4，最后取出来8，对应代码：
```php
$lb = $mi - $lo; //$b数组的边界
$lc = $hi - $mi; //$c数组的边界
$res = [];
//$i表示合并后数组的下标 $ib是b数组的下标 $ic是c数组的下标 
for($i = 0,$ib=0,$ic=0;$ib<$lb || $ic < $lc;){
    //ib 下标没有越界 && c的数组已经空了也就是$ic >= $lc || 比较两个数组首位的大小 如果b的首元素 < c的首元素，那么取出来b的首元素
    if ($ib < $lb && ( $ic >= $lc || $b[$ib] <= $c[$ic])) {
        $res[$i++] = $b[$ib++];
    }
    //k 下标没有越界 && b的数组已经空了也就是$ib >= $lb || 如果c的首元素 < b的首元素，那么取出来c的首元素 
    if ($ic < $lc && ($ib >= $lb || $b[$ib] > $c[$ic])) {
        $res[$i++] = $c[$ic++];
    }
}
return $res;
```

示意图如下：
!(mpdf)[../images/mergesort02.png]


