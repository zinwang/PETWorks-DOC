+++++++++++++++++++++++++++++++++++++++
資料 t-相似性評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估資料是否滿足 t-相似性（t-closeness）。更多 t-相似性之說明，詳見 `此處 <#id3>`_ 。

我們以 ``data/patient_anonymized.csv`` 作為去識別化資料表， ``data/patient_hierarchy`` 作為
資料階層定義，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework
框架判斷此指標。

在以下程式碼中，我們透過 API ``PETValidation(None, anonymized, "t-closeness", dataHierarchy, attributeTypes, tLimit)`` ，以上述資料、“t-closeness” 字串與 ``tLimit`` 作爲
參數，判斷資料是否滿足 t-相似性。

再來，我們透過 API ``report(result, format)`` ，以上述評估結果與 “json” 字串作爲參數，將評估
結果以 JSON 格式印出。


範例程式碼: t-closeness.py
--------------------------

.. code-block:: python

   from PETWorks import PETValidation, report
    from PETWorks.attributetypes import (
        SENSITIVE_ATTRIBUTE,
        QUASI_IDENTIFIER,
    )

    anonymized = "data/patient_anonymized.csv"
    dataHierarchy = "data/patient_hierarchy"

    attributeTypes = {
        "ZIPCode": QUASI_IDENTIFIER,
        "Age": QUASI_IDENTIFIER,
        "Disease": SENSITIVE_ATTRIBUTE,
    }

    result = PETValidation(
        None,
        anonymized,
        "t-closeness",
        dataHierarchy=dataHierarchy,
        attributeTypes=attributeTypes,
        tLimit=0.376,
    )
    report(result, "json")


輸出結果
--------

.. code-block:: text
    
    $ python3 t-closeness.py
    {
        "t": 0.376,
        "fulfill t-closeness": true
    }

評估指標之定義
--------------

t-相似性（t-closeness）是一種隱私保護力指標，表示資料集中子群體的敏感資訊機率分佈與資料表整體的敏感資訊機率分佈之差異，是否不超過 t，該差異稱為推土機距離（Earth Mover’s distance）。一旦攻擊者發現子群體的敏感資訊機率分佈與整體資料表差異過大，可藉由資料表推敲出子群體的敏感資訊特徵。

推土機距離數值分佈在 0 到 1 之間。當數值越高，子群體的敏感資訊機率分佈與整體的敏感資訊機率分佈越迥異，攻擊者可分析出的特徵越多；當數值越低，子群體的敏感資訊機率分佈與整體的敏感資訊機率分佈越相似，攻擊者可分析出的特徵越少。

因此，指標使用可調配參數 t 限制推土機距離最大值。當參數 t 越小，子群體的敏感資訊機率分佈與整體的敏感資訊分佈被限制的越相似，攻擊者可分析出的特徵越少，敏感資訊得到的保障越大。

評估指標之判斷
---------------

參考 [1]_ ，判斷 t-相似性的方法有四個步驟：

1. 將具有相同個體背景資訊之資料列分為同組。
2. 盤點各組別相較於整體資料表的敏感資訊數值之出現機率差異。
3. 計算將出現機率差異中和的最小成本，即推土機距離。
4. 確認上述推土機距離是否不超過 t 。


接下來將以下列「範例資料表」與「疾病資料階層」說明判斷過程。


**範例資料表**

+---------------------+-------------------------+--------------------------------+--------------------------------+
| .. centered::  編號 | .. centered::  出生年   | .. centered::  薪水            | .. centered::  疾病            | 
+=====================+=========================+================================+================================+
| .. centered::  1    | .. centered::  197*     | .. centered::  3k              | .. centered::  胃癌            | 
+---------------------+-------------------------+--------------------------------+--------------------------------+
| .. centered::  2    | .. centered::  197*     | .. centered::  4k              | .. centered::  流感            |  
+---------------------+-------------------------+--------------------------------+--------------------------------+
| .. centered::  3    | .. centered::  198*     | .. centered::  5k              | .. centered::  流感            |  
+---------------------+-------------------------+--------------------------------+--------------------------------+
| .. centered::  4    | .. centered::  198*     | .. centered::  6k              | .. centered::  胃炎            |  
+---------------------+-------------------------+--------------------------------+--------------------------------+
| .. centered::  5    | .. centered::  198*     | .. centered::  8k              | .. centered::  胃癌            |  
+---------------------+-------------------------+--------------------------------+--------------------------------+

**疾病資料階層**

.. image:: https://i.imgur.com/HF8XGrq.png


假設「範例資料表」各欄位之屬性型態如下：

1. 「出生年」為個體背景資訊。
2. 「薪水」為數字類型敏感資訊。
3. 「疾病」為非數字類型敏感資訊。

假設「疾病」欄位之數值可依「疾病資料階層」如下分類：

1. 「疾病」下可細分「呼吸感染」與「胃部疾病」。
2. 「呼吸感染」下可細分「流感」。
3. 「胃部疾病」下可細分「胃癌」與「胃炎」。


若欲判斷「範例資料表」是否滿足 t = 4 之 t-相似性，可以如下進行。

**STEP 1：將具有相同個體背景資訊之資料列分為同組。** 

以「範例資料表」為例，將具有相同「出生年」個體背景資訊之資料列分組，可得兩組如下：

1. 「出生年」資料為「197*」之組別，包含 **編號 1、2** 兩個資料列。
2. 「出生年」資料為「198*」之組別，包含 **編號 3、4、5** 三個資料列。


**STEP 2：盤點各組別相較於整體資料表的敏感資訊數值之出現機率差異。** 

出現機率差異公式如下，

.. math::
    \begin{equation}
    \begin{aligned}
    & 組別中敏感資訊數值出現機率差異 \\
    & = 組別中敏感資訊數值出現機率 - 整體資料表敏感資訊數值出現機率 \\ 
    &= \frac{組別中敏感數值出現次數}{組別資料列數} - \frac{整體資料表敏感數值出現次數}{整體資料表資料列數}
    \end{aligned}
    \end{equation}

以 **編號 1、2 資料列** 組別中，**「薪水」資料** 之 **「5k」數值** 的出現機率差異為例。其在組別中出現 0 次，因此「5k」於組別之出現機率為 0；其在「範例資料表」中出現 1 次，且「範例資料表」有 5 個資料列，因此「5k」於「範例資料表」之出現機率為 1/5。故，數值「5k」的出現機率差異為 **-1/5**。

.. math ::

    \begin{equation}
    \begin{aligned}
    & 編號 1、2 資料列組別中「5k」出現機率差異 \\ 
    & = 編號 1、2 資料列組別中「5k」出現機率 - 「範例資料表」「5k」出現機率 \\ 
    & = \frac{編號 1、2 資料列組別中「5k」出現次數}{編號 1、2 資料列組別資料列數} - \frac{「範例資料表」「5k」出現次數}{「範例資料表」資料列數} \\
    & = \frac{0}{2} - \frac{1}{5} = -\frac{1}{5}
    \end{aligned}
    \end{equation}


依同樣邏輯計算，可得 **編號 1、2 資料列** 組別之「薪水」、「疾病」資料相較於「範例資料表」的敏感數值出現機率差異，如下表所示：

+-----------------------------+--------------------+--------------------+---------------------+---------------------+---------------------+
| .. centered:: 「薪水」數值  | .. centered:: 3k   | .. centered:: 4k   | .. centered:: 5k    | .. centered:: 6k    | .. centered:: 8k    | 
+=============================+====================+====================+=====================+=====================+=====================+
| .. centered:: 出現機率差異  | .. centered:: 3/10 | .. centered:: 3/10 | .. centered::  -1/5 | .. centered::  -1/5 | .. centered::  -1/5 | 
+-----------------------------+--------------------+--------------------+---------------------+---------------------+---------------------+

+-------------------------------+-------------------------+------------------------+-------------------------+
| .. centered:: 「疾病」數值    | .. centered::  胃癌     | .. centered::  流感    | .. centered::  胃炎     | 
+===============================+=========================+========================+=========================+
| .. centered:: 出現機率差異    | .. centered::  1/10     | .. centered::  1/10    | .. centered::  -1/5     | 
+-------------------------------+-------------------------+------------------------+-------------------------+


**STEP 3：計算將出現機率差異中和的最小成本，即推土機距離。** 

針對 **數字類型** 與 **非數字類型** 兩種敏感資訊，中和出現機率差異的最小成本有兩種不同計算方式，以下分別說明之。

**一、數字類型敏感資訊之中和最小成本**

針對數字類型敏感資訊，欲計算此類型之中和最小成本，需根據敏感數值的大小，逐步將出現機率差異由小轉移到更大的數值上，並將每次轉移的成本加總得之。單次轉移成本公式如下，

.. math :: 
    \begin{equation}
    \begin{aligned}
    單次轉移成本 &= \frac{1}{整體資料表敏感數值種類數 - 1} \times 出現機率之轉移量
    \end{aligned}
    \end{equation}

以 **編號 1、2 資料列** 組別中，**「薪水」資料** 數值為例，總共需要轉移出現機率差異 4 次，分別將出現機率差異由「3k」至「4k」、「4k」至「5k」、「5k」至「6k」與「6k」至 「8k」依序進行轉移，轉移成本總和即為中和最小成本。



.. math :: 
    \begin{equation}
    \begin{aligned}
    3k \rightarrow 4k \rightarrow 5k \rightarrow 6k \rightarrow 8k
    \end{aligned}
    \end{equation}



以「3k」至「4k」之轉移為例，初始狀態下「3k」出現機率差異為 3/10，因此出現機率轉移量為 **3/10** 、轉移成本為 3/40，轉移後「4k」之出現機率差異為 3/10 + 3/10 = 6/10。


.. math :: 
    \begin{equation}
    \begin{aligned}
    &「3k」轉移出現機率差異至「4k」之成本 \\ 
    & = \frac{1}{「範例資料表」敏感數值種類數 - 1} \times 出現機率之轉移量 \\
    & = \frac{1}{5 - 1} \times \frac{3}{10} = 
    \frac{3}{40}
    \end{aligned}
    \end{equation}


以「4k」至「5k」之轉移為例，前次轉移後「4k」出現機率差異為 6/10，因此出現機率轉移量為 **6/10** 、轉移成本為 6/40，轉移後「5k」之出現機率差異為 (-1/5) + 6/10 = 4/10。


.. math :: 
    \begin{equation}
    \begin{aligned}
    &「4k」轉移出現機率差異至「5k」之成本 \\ 
    & = \frac{1}{「範例資料表」敏感數值種類數 - 1} \times 出現機率之轉移量 \\
    & = \frac{1}{5 - 1} \times \frac{6}{10} = 
    \frac{6}{40}
    \end{aligned}
    \end{equation}

依同樣邏輯計算，可得 **編號 1、2 資料列** 組別 4 次轉移成本分別為 3/40 、 6/40 、 4/40 與 2/40，加總得出中和「薪水」資料之最小成本為 **3/8** 。

**二、非數字類型敏感資訊之中和最小成本**

針對非數字類型敏感資訊，欲計算此類型之中和最小成本，需根據資料階層的高度，逐步由低至高正負相消資料階層各個內部節點下的出現機率差異，並將每次相消的成本加總得之。單次正負相消成本公式如下，


.. math :: 
    \begin{equation}
    \begin{aligned}
    單次正負相消成本 &= \frac{內部節點於資料階層高度}{資料階層總高度} \times 出現機率之正負相消量
    \end{aligned}
    \end{equation}


以 **編號 1、2 資料列** 組別中，**「疾病」資料** 數值為例，總共需要正負相消出現機率差異 3 次，分別由低至高對於「呼吸感染」、「胃部疾病」、「疾病」底下的出現機率差異進行正負相消，相消成本總和即為中和最小成本。

以「呼吸感染」內部節點為例，初始狀態下其底下出現機率差異正值總和為 1/10，負值總和為 0，無法正負相消，故相消成本為 **0** ，「呼吸感染」繼承其底下出現機率差異正值，1/10。


以「胃部疾病」內部節點為例，初始狀態下其底下出現機率差異正值總和為 1/10，負值總和為 -1/5 ，出現機率之正負相消量為 **1/10** ，且資料階層高度為 **1** ，故相消成本為 1/20，相消完「胃部疾病」繼承剩餘之出現機率差異， 1/10 + (-1/5) = 1/20。

.. math :: 
    \begin{equation}
    \begin{aligned}
    &「胃部疾病」節點正負相消之成本 \\
    & = \frac{「胃部疾病」節點於資料階層高度}{資料階層總高度} \times 出現機率之正負相消量\\
    & = \frac{1}{2} \times \frac{1}{10}  = \frac{1}{20}
    \end{aligned}
    \end{equation}

以「疾病」內部節點為例，經前次相消其底下節點出現機率差異正值總和為 1/10，負值總和為 -1/10 ，出現機率之正負相消量為 **1/10** ，且資料階層高度為 **2** ，故相消成本為 1/10 ，相消完「疾病」繼承剩餘之出現機率差異，1/10 + (-1/10) = 0。

.. math :: 
    \begin{equation}
    \begin{aligned}
    &「疾病」節點正負相消之成本 \\
    & = \frac{「疾病」節點於資料階層高度}{資料階層總高度} \times 出現機率之正負相消量\\
    & = \frac{2}{2} \times \frac{1}{10}  = \frac{1}{10}
    \end{aligned}
    \end{equation}



於是可得 **編號 1、2 資料列** 組別所有相消成本分別為 0、1/20 與 1/10，加總得出中和「疾病」資料之最小成本為 **3/20** 。



**STEP 4：確認上述推土機距離是否不超過 t 。**

最後，確認步驟 3 計算之推土機距離，即中和最小成本，是否不超過 t 。在此例中，**編號 1、2 資料列** 與 **編號 3、4、5 資料列** 組別的 **「薪水」** 與 **「疾病」** 資料推土機距離如下表，數值皆不超過 4 。因此，「範例資料表」滿足 t = 4 之 t-相似性。


+---------------------------------------+-------------------------------------+--------------------------------------+
| .. centered:: 組別                    | .. centered:: 「薪水」資料推土機距離| .. centered:: 「疾病」資料推土機距離 |
+=======================================+=====================================+======================================+
| .. centered:: **編號 1、2 資料列**    | .. centered:: 3/8                   | .. centered:: 1/4                    | 
+---------------------------------------+-------------------------------------+--------------------------------------+
| .. centered:: **編號 3、4、5 資料列** | .. centered:: 3/20                  | .. centered:: 1/10                   |
+---------------------------------------+-------------------------------------+--------------------------------------+

參考資料
---------

.. [1] N. Li, T. Li and S. Venkatasubramanian, “t-Closeness: Privacy Beyond k-Anonymity and l-Diversity,” 2007 IEEE 23rd International Conference on Data Engineering, Istanbul, Turkey, 2007, pp. 106-115, doi: 10.1109/ICDE.2007.367856.
