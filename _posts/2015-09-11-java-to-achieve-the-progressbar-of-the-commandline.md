---
layout: post
title: Java实现命令行的进度条
categories: java
tags: java
---

```java
public class ProgressBar {
	private double finishPoint;
	private double barLength;
	
	public ProgressBar(){
		this.finishPoint = 100;
		this.barLength = 20;
	}
	
	public ProgressBar(double finishPoint, int barLength){
		this.finishPoint = finishPoint;
		this.barLength = barLength;
	}
	
	/**
	 * 显示进度条
	 * @param currentPoint 当前点
	 * @param finishPoint 结束点
	 * @param barLength 进度条长度(字符)
	 * @return 进度条结果
	 */
	public void showBarByPoint(double currentPoint) {
		// 根据进度参数计算进度比率
		double rate = currentPoint / this.finishPoint;
		// 根据进度条长度计算当前记号
		int barSign = (int) (rate * this.barLength);
		// 生成进度条
		System.out.print("\r");
		System.out.print(makeBarBySignAndLength(barSign) + String.format(" %.2f%%", rate * 100));
	}
	
	/**
	 * 构造进度条
	 * @param barSign 进度条标记(当前点)
	 * @param barLength 进度条长度
	 * @return 字符型进度条
	 */
	private String makeBarBySignAndLength(int barSign) {
		StringBuilder bar = new StringBuilder();
		bar.append("[");
		for (int i=1; i<=this.barLength; i++) {
			if (i < barSign) {
				bar.append("-");
			} else if (i == barSign) {
				bar.append(">");
			} else {
				bar.append(" ");
			}
		}
		bar.append("]");
		return bar.toString();
	}
}
```