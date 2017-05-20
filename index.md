<p dir='rtl' align='right'>بسم الله الرحمن الرحیم</p>

<p dir = 'rtl' align= 'right'> 
یکی از قابلیت های پایتون، Decorator ها می‌باشند که به صورت پوشش یک تابع یا کلاس عمل می‌کنند و رفتار کد را قبل و بعد از اجرای تابع تغییر می‌دهند بدون اینکه نیازی به تغییر خود تابع باشد. برای مثال در کد زیر با نوشتن @p_decorate قبل تابع موردنظر get_text قصد تغییری در آن داریم، با اینکار درواقع این تابع را به تابع p_decorator پاس داده‌ایم و در این تابع، تابع func_wrapper که تابع پاس داده‌شده را اجرا و ویرایش می‌کند، را به خروجی می‌دهیم. در عمل به جای تابع قبلی، تابع func_wrapper اجرا می‌شود که از تابع قبلی استفاده می‌کند.
</p>

```
def p_decorate(func):
   def func_wrapper(name):
       return "<p>{0} </p>". format(func(name))
   return func_wrapper

@p_decorate
def get_text(name):
   return "lorem ipsum, {0} dolor sit amet". format(name)

print get_text("John")

# Outputs <p>lorem ipsum, John dolor sit amet</p>
```

<p dir = 'rtl' align= 'right'> 
در صورتی که از چندین Decorator استفاده کنیم ترتیبشان قبل تابع مهم است و از نزدیکترین به تابع اصلی به دورترین اجرا می‌شود. برای استفاده‌کردن Decorator در متدهای داخل کلاس نیز از args  و *kwargs به عنوان پارامتر برای تابع خروجی داده‌شده (func_wrapper) استفاده می‌کنیم که باعث می‌شود، این تابع هر تعداد پارامتر را قبول کند. تابع wrapperکه به جای تابع اصلی ما اجرا می‌شود باعث می‌شود نوع __name__ ، __doc__ و __module__ تابع اصلی ما تغییر کند، که درعیب یابی کد، ما را دچار مشکل می‌کند، لذا میتوانیم از ماژول functools از تابع @wraps(func) به عنوان Decorator تابع خروجی داده شده (func_wrapper) استفاده کنیم. که در کد زیر نحوه استفاده آمده است:
</p>


```
from functools import wraps
def p_decorate(func):

   @wraps(func)
   def func_wrapper(*args, **kwargs):
       return "<p>{0}</p>".format(func(*args, **kwargs))
   return func_wrapper

class Person(object):
    def __init__(self):
        self.name = "John"
        self.family = "Doe"

    @p_decorate
    def get_fullname(self):
        return self.name+" "+self.family

my_person = Person()

print my_person.get_fullname()  #<p> John Doe</p>
print my_person.get_fullname.__name__ # get_fullname
print my_person.get_fullname.__doc__ # None
print my_person.get_fullname.__module__ # __main__
```


<p dir = 'rtl' align= 'right'> 
یکی از کاربردهای Decorator در دادن سطح دسترسی انواع کاربر برای توابع مختلف می‌باشد. به این منظور که برخی کاربران که مثلا وارد سیستم شده‌اند (Logged_in )، اجازه اجرای برخی توابع را خواهند داشت. به این منظور میتوانیم از decoratorای مثل @reguires_logged_in که در مثال زیر آمده استفاده کنیم .کسانی قادر به اجرای new_game را خواهد داشت که چنین سطح دسترسی داشته باشند در غیر اینصورت با Exception روبرو خواهند شد.
</p>


```
def requires_logged_in(fn):
    def ret_fn(*args,**kwargs):
        lPermissions = get_permissions(current_user_id())
        if 'logged_in' in lPermissions:
            return fn(*args,**kwargs)
        raise Exception("Not allowed")
    return ret_fn

@requires_logged_in 
def new_game():
    """
    any logged in user can start a new game
    """
```


<p dir = 'rtl' align= 'right'> 
اگر دو سطح دسترسی دیگر مثل administrator و premium_member نیز داشته باشیم، Decorator ها شبیه کد بالا خواهد بود، لذا جهت جلوگیری از تکرار، میتوانیم پاس دادن، داده به Decorator کمک بگیریم. برای اینکار، باید از تابعی استفاده کنیم که این داده را بگیرد و Decorator  را برگرداند همانند کد زیر عمل می‌کنیم که تابع reguires_permission  نوع کاربر که اجازه اجرا دارد را گرفته و پس از فراخوانی توابع decorator و decorated ،(به علت مفهموم Closure این توابع به sPermission دسترسی دارند.) با نوع کاربری که در حال اجراست مقایسه میکند و یا تابع را اجرا و یا Exception می‌دهد. ما به این دلیل از سه سطح تابع استفاده میکنیم که اولا باید مقدار برگشتی تابع اول، تابعی باشد که شامل تابع decorated شده باشد و تابع دوم نیز تابعی که باید جای تابع اصلی جایگذاری شود، را برگرداند.
</p>


```
def requires_permission(sPermission):                            
    def decorator(fn):                                            
        def decorated(*args,**kwargs):                            
            lPermissions = get_permissions(current_user_id())     
            if sPermission in lPermissions:                       
                return fn(*args,**kwargs)                         
            raise Exception("permission denied")                  
        return decorated                                          
    return decorator       
    
#and now we can decorate stuff...                                     

@requires_permission('administrator')
def delete_user(iUserId):
   #delete the user with the given Id. This function is only accessible to users with administrator permissions
   

@requires_permission('logged_in')
def new_game():
    #any logged in user can start a new game
    
@requires_permission('premium_member')
def premium_checkpoint():
   #save the game progress, only accessable to premium members
```


<p dir = 'rtl' align= 'right'> 
درنهایت فرم کلی تابع Decorator به صورت مقابل می‌باشد:
</p>


```
def outer_decorator(*outer_args,**outer_kwargs):                            
    def decorator(fn):                                            
        def decorated(*args,**kwargs):                            
            do_something(*outer_args,**outer_kwargs)                      
            return fn(*args,**kwargs)                         
        return decorated                                          
    return decorator      
```

<p dir = 'rtl' align= 'right'> 
منابع:
</p>

[1] ---, A guide to Python’s function decorators, https://www.thecodeship.com/patterns/guide-to-python-function-decorators/, May 20, 2017.

[2] ---, Advanced Uses of Python Decorators, https://www.codementor.io/sheena/advanced-use-python-decorators-class-function-du107nxsv, May 20, 2017.

[3] Bruce Eckel, Python Decorators II: Decorator Arguments, http://www.artima.com/weblogs/viewpost.jsp?thread=240845, May 20, 2017.
