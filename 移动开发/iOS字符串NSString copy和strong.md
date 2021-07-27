iOS字符串NSString copy和strong


这主要是为了防止NSString被修改。当NSString的赋值来源也是NSString时，strong和copy的作用相同，都是给复制来源的引用计数加1；当NSStrig的赋值来源是NSMutableString时，copy会做深拷贝，即重新生成一个新的对象，修改赋值来源不会影响NSString的值


```
@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    NSMutableString *s = [NSMutableString stringWithString:@"name"];
    NSLog(@"原始字符串在内存中地址:%p",s);
    
    A *a = [[A alloc] init];
    a.testString = s;
    NSLog(@"A对象字符串在内存中地址:%p",a.testString);
    [a.testString appendString:@"abc"];
    NSLog(@"a:%@",a.testString);
    
    B *b = [[B alloc] init];
    b.testString = a.testString;
    NSLog(@"B对象字符串在内存中地址:%p",b.testString);
    NSLog(@"b:%@",b.testString);
    
    NSMutableString *string = [NSMutableString stringWithString:@"name"];
    NSLog(@"原始字符串string在内存中地址:%p",string);
    
    NSMutableString *copyString = [string copy];
    NSLog(@"copy字符串string在内存中地址:%p",copyString);
    
    //[copyString appendString:@"test"];
    
    NSString *string1 = [NSString stringWithFormat:@"name"];
    NSLog(@"原始string的地址:%p",string1);
    
    NSString *copyString1 = [string1 copy];
    NSLog(@"拷贝string的地址:%p",copyString1 );
    
    IGTrackManager *mg = [[IGTrackManager alloc] init];
    [mg testPrint:@"track mg:"];
    
    [[[IGNetworkManager alloc] init] testPrint:@"net"];
}


@end
```

A类
```
@interface A : NSObject
@property (nonatomic, strong) NSMutableString *testString;
@end

@implementation A

@end
```

B类
```
@interface B : NSObject
@property (nonatomic, copy) NSMutableString *testString;
@end

@implementation B

@end

```
