---
layout: post
title: JFreeChart in Eclipse RCP & SWT
published: true
tags: java, eclipse, swt
comments: true
---

To use JFreeChart in a Eclipse RCP project or simply embed in an SWT composite, you have two applicable choices:
 `org.jfree.experimental.chart.swt.ChartComposite` from JFreeChart SWT experimental project, which I can not say works seamless.

SWT_AWT bridge, hmm, not perfect but plays its role better.

- Create a container composite with `SWT.EMBEDED` style (required for `SWT_AWT`).

```java
Composite container = new Composite(parent, SWT.EMBEDDED)
```

- Create new AWT Frame using container.

```java
java.awt.Frame chartFrame = SWT_AWT.new_Frame(container)
```

- Create a JFreeChart `ChartPanel` and append on chartFrame. (This should be done in AWT Event Thread)

```java
SwingUtilities.invokeLater(new Runnable() {
     public void run() {
          ChartPanel chartPanel = new ChartPanel(chart, true);
          chartFrame.add(chartPanel);
     }
})
```

That's it!
