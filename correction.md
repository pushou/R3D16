<!-- %        https://qmacro.org/blog/posts/2022/05/19/json-object-values-into-csv-with-jq/

%        ## curl des prix region us-east-1
%        ```
%        curl -H 'accept: json' 'http://localhost:6001?region=us-east-1&filter=m5,m6&sort=vCPUs'|jq '.' m5.json
%        ```
       
       
%        ## Pour les clefs du csv
       
%        ```
%        > jq -r '.Prices[]| [.InstanceType, .Memory, .VCPUS, .Storage, .Network, .Price, .MonthlyPrice, .SpotPrice]\|@csv' m5.json   
%        ```
       
       
%        ```
%        jq -r '.Prices[]| keys|@csv' medium.json
%        "Cost","InstanceType","Memory","MonthlyPrice","Network","SpotPrice","Storage","VCPUS"
%        "Cost","InstanceType","Memory","MonthlyPrice","Network","SpotPrice","Storage","VCPUS"
%        "Cost","InstanceType","Memory","MonthlyPrice","Network","SpotPrice","Storage","VCPUS"
%        "Cost","InstanceType","Memory","MonthlyPrice","Network","SpotPrice","Storage","VCPUS"
%        ```
       
%        ## juste les champs . On sélectionne la première ligne, on récupère les clefs dans $k et on affiche $k
       
%        ```
%         jq -r '.Prices[0]| keys' medium.json
%        [
%          "Cost",
%          "InstanceType",
%          "Memory",
%          "MonthlyPrice",
%          "Network",
%          "SpotPrice",
%          "Storage",
%          "VCPUS"
%        ]
%        ❯ jq -r '.Prices[0]| keys[]' m5.json
%        Cost
%        InstanceType
%        Memory
%        MonthlyPrice
%        Network
%        SpotPrice
%        Storage
%        VCPUS
       
%        > jq -r '.Prices[0]| keys[] as $k|$k' m5.json
%        Cost
%        InstanceType
%        Memory
%        MonthlyPrice
%        Network
%        SpotPrice
%        Storage
%        VCPUS
       
%        >  jq -r '.Prices[0]| keys|@csv' m5.json
%        "Cost","InstanceType","Memory","MonthlyPrice","Network","SpotPrice","Storage","VCPUS"
       
       
%        ```
%        ## Pour les valeurs du csv
       
%        ### un moyen simple
       
%        ```
%        jq -r '.Prices[]| join(",")' medium.json 
       
%        m5ad.large,8 GiB,2,1 x 75 NVMe SSD,Up to 10 Gigabit,0.103,75.19,0.0344
%        m5n.4xlarge,64 GiB,16,EBS only,Up to 25 Gigabit,0.952,694.9599999999999,0.4110
%        m6g.2xlarge,32 GiB,8,EBS only,Up to 10 Gigabit,0.308,224.84,0.1696
%        m6i.metal,512 GiB,128,EBS only,50000 Megabit,6.144,4485.12,2.2852
%        m6id.24xlarge,384 GiB,96,4 x 1425 SSD,37500 Megabit,5.6952,4157.496,1.7139
%        m5dn.2xlarge,32 GiB,8,1 x 300 NVMe SSD,Up to 25 
%        ...
%        ### moyen plus complexe
       
%        #### Récupération des valeurs
       
%        ```
%        jq -r '.Prices[]| [.InstanceType, .Memory, .VCPUS, .Storage, .Network, .Price, .MonthlyPrice, .SpotPrice]' m5.json|jq
       
       
%        [
%          [
%            "m6i.2xlarge",
%            "32 GiB",
%            8,
%            "EBS only",
%            "Up to 12500 Megabit",
%            null,
%            280.32,
%            "0.1759"
%          ],
%          [
%            "m5a.8xlarge",
%            "128 GiB",
%            32,
%            "EBS only",
%            "Up to 10 Gigabit",
%            null,
%            1004.4799999999999,
%            "0.5915"
%        ...
       
%        On peut aussi utiliser map
%        ```
%        jq -r '.Prices| map([.InstanceType, .Memory, .VCPUS, .Storage, .Network, .Price, .MonthlyPrice, .SpotPrice])' m5.json
%        ```
       
       
%        ##### passage au csv
%        ```
%        jq -r '.Prices[]| [.InstanceType, .Memory, .VCPUS, .Storage, .Network, .Price, .MonthlyPrice, .SpotPrice]|@csv' m5.json
%        ```
       
%        ```
%        jq -r '.Prices| map([.InstanceType, .Memory, .VCPUS, .Storage, .Network, .Price, .MonthlyPrice, .SpotPrice])|.[]|@csv' m5.json
%        ```
       
       
%        ### formes générique
       
%        ```
%        jq -r '.Prices[] | [keys[] as $k | .[$k]]|@csv' m5.json 
       
%        0.384,"m6i.2xlarge","32 GiB",280.32,"Up to 12500 Megabit","0.1759","EBS only",8
%        1.376,"m5a.8xlarge","128 GiB",1004.4799999999999,"Up to 10 Gigabit","0.5915","EBS only",32
%        6.528,"m5dn.24xlarge","384 GiB",4765.44,"100 Gigabit","1.6941","4 x 900 NVMe SSD",96
%        0.904,"m5d.4xlarge","64 GiB",659.9200000000001,"Up to 10 Gigabit","0.3574","2 x 300 NVMe SSD",16
%        0.077,"m6g.large","8 GiB",56.21,"Up to 10 Gigabit","0.0464","EBS only",2
%        0.1808,"m6gd.xlarge","16 GiB",131.98399999999998,"Up to 10 Gigabit","0.0714","1 x 237 NVMe SSD",4
%        1.648,"m5ad.8xlarge","128 GiB",1203.04,"Up to 10 Gigabit","0.7226","2 x 600 NVMe SSD",32
%        0.226,"m5d.xlarge","16 GiB",164.98000000000002,"Up to 10 Gigabit","0.0776","1 x 150 NVMe SSD",4
%        0.192,"m5.xlarge","16 GiB",140.16,"Up to 10 Gigabit","0.0686","EBS only",4
%        2.1696,"m6gd.12xlarge","192 GiB",1583.808,"20 Gigabit","0.8570","2 x 1425 NVMe SSD",48
%        7.5936,"m6id.32xlarge","512 GiB",5543.328,"50000 Megabit","2.2852","4 x 1900 SSD",128
%        0.113,"m5d.large","8 GiB",82.49000000000001,"Up to 10 Gigabit","0.0360","1 x 75 NVMe SSD",2
%        5.712,"m5n.metal","384 GiB",4169.76,"100 Gigabit","1.6323","EBS only",96
%        0.616,"m6g.4xlarge","64 GiB",449.68,"Up to 10 Gigabit","0.3584","EBS only",16
%        0.096,"m6i.large","8 GiB",70.08,"Up to 12500 Megabit","0.0366","EBS only",2
%        0.119,"m5n.large","8 GiB",86.86999999999999,"Up to 25 Gigabit","0.0340","EBS only",2
%        ...
       
%        ```
%        Redirection vers le csv
%        ```
%        jq -r '.Prices[] | [keys[] as $k | .[$k]]|@csv' m5.json|less>> m5.csv
%        ``` -->