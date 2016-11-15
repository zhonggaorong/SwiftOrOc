# SwiftOrOc
swift与OC的混编，在swift工程中引入oc类，并且使用OC类中的代理与Block.


最新一些学妹问起，所以抽点时间来写的，适合入门级别的swift 与 OC 混编 的程序猿。  

本文章将从两个方向分别介绍 OC 与 swift 混编  


1. 第一个方向从 swift工程 中引入 oc类 

    1. 1 如何在swift的类中使用oc类
    1.2  如何在swift中实现oc的代理方法
    1.3   如何在swift中实现oc的Block回调

2. 第二个方向从OC工程中引入swift类

    2.1  如何在OC类中使用swift类
    2.2   如何在OC中实现swift的代理方法
    2.3   如何在OC中实现swift中类似Block回调


下面是具体的实现过程：
 1.1  如何在swift的类中使用oc类？ 

1.  swift工程中引入OC类。 具体实现过程。
    1.1 新建一个swift工程类。 取名 swiftOrOC
    1.2  实现的功能为 ：  从swift. viewController.swift 中 push到 OC语言 secondViewController 控制器
	1.2.1  新建SecondViewController 类 。
        
    	1.2.2 建立桥接文件。 （很重要）
			

    一定要记得点击这个按钮。 
       1.2.3  接下来工程目录如下：
	
       
     1.2.4 接下来就可以实现具体的跳转功能了。 
      ViewController.swift中具体实现
     
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var hintLabel: UILabel!  //稍后用来显示回调
    
    // push 到 oc controller
    @IBAction func pushAction(_ sender: AnyObject) {
        let secondVC = SecondViewController.init()
        self.navigationController?.pushViewController(secondVC, animated: true)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }


}


1.2 如何在swift中实现oc的代理方法

       1.2.1 首先在 SecondViewController.h 中声明一个协议。具体代码
        
#import <UIKit/UIKit.h>

@protocol SecondDelegate <NSObject>

-(void)refreshHintLabel:(NSString *)hintString;

@end

@interface SecondViewController : UIViewController

@property (nonatomic,weak)id<SecondDelegate> secondDelegate;
@end
     1.2.2 然后在SecondViewController.m中，通过一个UITextField，让用户输入内容，当用户点击返回的时候把输入框中的内容返回给对应的代理。具体代码如下
    
#import "SecondViewController.h"
#import "UIViewController+BackButtonHandler.h"

@interface SecondViewController ()
{
    UITextField *textField;
}
@end

@implementation SecondViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.title = @"oc";
    
    self.view.backgroundColor  = [UIColor whiteColor];
    
    textField = [[UITextField alloc]initWithFrame:CGRectMake(100, 100, 200, 200)];
    textField.placeholder = @"请输入用户名";
    [self.view addSubview:textField];
    [textField.layer setBorderColor:[UIColor blackColor].CGColor];
    [textField.layer setBorderWidth:1.0];

    
}

-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    [self.view endEditing:YES];
}
#pragma mark 返回上一页回调 ,将用户输入的用户名传回给 ViewController.swift
-(BOOL)navigationShouldPopOnBackButton{
    if ([_secondDelegate respondsToSelector:@selector(refreshHintLabel:)]) {
        [_secondDelegate refreshHintLabel: textField.text];
    }
    
    return YES;
}


- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

/*
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    // Get the new view controller using [segue destinationViewController].
    // Pass the selected object to the new view controller.
}
*/

@end
 	1.2.3 接下来就非常简单了，让ViewController.swift只需要成为SecondViewController的代理，然后遵循她的协议，就可以了。 具体代码如下。
       1.2.3.1 遵循协议
  
     1.2.3.2 成为代理，并实现协议方法，更改controller.swift中hintLabel的text。
    // push 到 oc controller
    @IBAction func pushAction(_ sender: AnyObject) {
        let secondVC = SecondViewController.init()
        secondVC.secondDelegate = self;
        self.navigationController?.pushViewController(secondVC, animated: true)
    }
    
    // SecondViewControll的代理方法
    func refreshHintLabel(_ hintString: String!) {
        hintLabel.text = "secondView textView.text = " + hintString;
    }
    

 1.3   如何在swift中实现oc的Block回调

	1.3.1 具体过程与1.2小节一样。 直接上代码。
        1.3.2 声明block;
         
typedef void(^RefreshHintLabelBlock)(NSString *hintString);

@interface SecondViewController : UIViewController
@property (nonatomic, copy) RefreshHintLabelBlock hintBlock;
@end

        1.3.3 block的回调。 SecondViewController.m中
#pragma mark 返回上一页回调 ,将用户输入的用户名传回给 ViewController.swift
-(BOOL)navigationShouldPopOnBackButton{    
    if (_hintBlock) {
        _hintBlock(textField.text);
    }
    return YES;
}

        1.3.4 在swift类中调用 oc的block.
    // push 到 oc controller
    @IBAction func pushAction(_ sender: AnyObject) {
        let secondVC = SecondViewController.init()
//        secondVC.secondDelegate = self;
        secondVC.hintBlock = {(t:String?)in
            self.hintLabel.text = "secondView textView.text = " + t!
        }
        self.navigationController?.pushViewController(secondVC, animated: true)
    }


   工程已上传到git上，git地址： 
2.  OC工程中引入swift类。 具体实现过程。
