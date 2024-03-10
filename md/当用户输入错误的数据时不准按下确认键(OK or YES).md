# 当用户输入错误的数据时不准按下确认键(OK or YES)

## 问题研究

创建一个Dialog，简单方法有

1. 通过JOptionPanel 的静态方法创建对话框
2. 创建一个JOptionPanel 对象，传递一个JPanel 对象
3. 直接继承JDialog对象。

1和2用户都不可以操作按钮，所以方法只有第三个。

## 过程

1. 新建一个Dialog，通过eclipse，这样大部分的代码都可以自动生成。结果是这样的

    ```java
        /**
        * Create the dialog.
        */
        public T1() {
            setBounds(100, 100, 450, 300);
            getContentPane().setLayout(new BorderLayout());
            contentPanel.setLayout(new FlowLayout());
            contentPanel.setBorder(new EmptyBorder(5, 5, 5, 5));
            getContentPane().add(contentPanel, BorderLayout.CENTER);
            {
                JPanel buttonPane = new JPanel();
                buttonPane.setLayout(new FlowLayout(FlowLayout.RIGHT));
                getContentPane().add(buttonPane, BorderLayout.SOUTH);
                {
                    JButton okButton = new JButton("OK");
                    okButton.setActionCommand("OK");
                    buttonPane.add(okButton);
                    getRootPane().setDefaultButton(okButton);
                }
                {
                    JButton cancelButton = new JButton("Cancel");
                    cancelButton.setActionCommand("Cancel");
                    buttonPane.add(cancelButton);
                }
            }
        }

    ```

    T1 是类名，仅截取了构造函数。
    看两个按钮的创建的部分，并没有添加事件（那个setActionCommand 不是的，不过在事件中需要这个），那就添加事件吧，比如下面这样

2. 添加事件

    ```java
        okButton.addActionListener(this);
    ```

    与

    ```java
        cancelButton.addActionListener(this);
    ```

    与

    ```java
        private int value;
        @Override
        public void actionPerformed(ActionEvent e) {
            if (e.getActionCommand().equals("OK")) {
                if (valid()) {
                    return;
                }
                value = JOptionPane.OK_OPTION;
            } else {
                value = JOptionPane.CANCEL_OPTION;
            }
            setVisible(false);
        }
    ```

    获得按钮的`actionCommand`，如果是`OK`，就进行判断，如果不通过就什么都不作（如果是以前此时也会关闭对话框，只能后续在通知用户输入的数据是错误的）。

    这个`valid()` 函数需要根据需要自由实现，用`value`存储结果，然后调用`setVisible(false)` 关闭对话框。