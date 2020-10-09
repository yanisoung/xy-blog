---
title: java swing  绘制简单的运动圆
date: 2019-11-18 23:25:03
categories:
- 编程
tags:
- java
keywords: java,game,游戏,swing
---
思考：
1.为什么我没有想到要提供canvasWidth和canvasHeight的get方法？而且只提供get方法？为什么不提供set方法？
    因为canvasWidth和canvasHeight是窗口的宽和高，给外部提供set方法之后，就可以被外部随意修改产生安全问题。
2.画板的部分为什么提供内部类？
    因为画板是这个窗口的一部分。

<!-- more -->

/**
 * 入口启动
 */
public static void main(String[] args) {

   int sceneWidth = 800;
    int sceneHeight = 800;
    int count = 10;

    MyCircleRun myCircleRun = new MyCircleRun(sceneWidth, sceneHeight, count);
    myCircleRun.run("我的圈圈");
}




/**
 * 启动类
 * @Author xu yan
 * @Date 2019-11-24 18:12
 */
public class MyCircleRun {
    /**
     * 窗口宽
     */
    private Integer sceneWidth;
    /**
     * 窗口高
     */
    private Integer sceneHeight;

    /**
     * 要绘制的数据
     */
    private Circle[] circles;

    public MyCircleRun(int sceneWidth, int sceneHeight, int count) {
        this.sceneHeight = sceneHeight;
        this.sceneWidth = sceneWidth;

        circles = new Circle[count];
        double r = 70;

        for (int i = 0; i < count; i++) {
            //sceneWidth - 2 * r  画布的宽-直径 作为随机取值的最大值  +r是为了不让圆在画布边缘随生成
            double x = (Math.random() * (sceneWidth - 2 * r)) + r;
            double y = (Math.random() * (sceneHeight - 2 * r)) + r;
            //随机生成速度
            double vx = (Math.random() * 11) - 5;
            double vy = (Math.random() * 11) - 5;

            circles[i] = new Circle(x, y, r, vx, vy);
        }
    }

    public void run(String windowTitle) {
        EventQueue.invokeLater(() -> {
            MyCircleJFrame frame = new MyCircleJFrame(windowTitle, sceneWidth, sceneHeight);

            new Thread(() -> {
                // TODO: 2019/11/24 为什么用循环？ 不停的重新绘制
                while (true) {
                    // 绘制数据
                    frame.render(circles);
                    AlgoVisHelper.pause(20);

                    // 更新数据（重新绘制后生效）
                    for (Circle circle : circles) {
                        circle.move(sceneWidth, sceneHeight);
                    }
                }
            }).start();
        });
    }
}

/**
 * 要绘制的类
 */
@Data
public class Circle {
    private double x;
    private double y;
    private double r;

    private double xv;
    private double yv;

    public Circle(double x, double y, double r, double xv, double yv) {
        this.x = x;
        this.y = y;
        this.r = r;
        this.xv = xv;
        this.yv = yv;
    }

    /**
     * 移动
     */
    public void move(int sceneWidth, int sceneHeight) {
        //碰到画布边缘自动弹回来
        //计算最小边缘
        double yMin = r;
        double xMin = r;
        //计算最大边缘
        double yMax = sceneHeight - r;
        double xMax = sceneWidth - r;
        //移动速度可以是负的 往反方向移动
        if (x >= xMax || x <= xMin) {
            xv = -xv;
        }
        if (y >= yMax || y <= yMin) {
            yv = -yv;
        }

        x += xv;
        y += yv;
    }

}


/**
 * 绘制工具类
 */

public class AlgoVisHelper {

    private AlgoVisHelper() {
    }

    public static void strokeCircle(Graphics2D g, double x, double y, double r) {

        Ellipse2D circle = new Ellipse2D.Double(x - r, y - r, 2 * r, 2 * r);
        g.draw(circle);
    }

    public static void fillCircle(Graphics2D g, int x, int y, int r) {

        Ellipse2D circle = new Ellipse2D.Double(x - r, y - r, 2 * r, 2 * r);
        g.fill(circle);
    }

    public static void setColor(Graphics2D g, Color color) {
        g.setColor(color);
    }

    public static void setStrokeWidth(Graphics2D g, int w) {
        int strokeWidth = w;
        g.setStroke(new BasicStroke(strokeWidth, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
    }

    /**
     * 绘制等待时间
     *
     * @param t
     */
    public static void pause(int t) {
        try {
            Thread.sleep(t);
        } catch (InterruptedException e) {
            System.out.println("Error sleeping");
        }
    }

}
