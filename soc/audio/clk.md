# CLK

## 时钟的产生

```
//fpga

MCLK --> 24Mhz
------
	 | BCLK = MCLK / T_BCLKDIV       	//24NHz / 8 = 3MHz
 	 |_____
		   | SCLK = BCLK / T_SYNC_DIV   //3MHz / 64 = 46.875KHz ~ 44.1KHz
		   |____
```

## 

