# TokenAutoCompleteXamarinBinding
Xamarin Binding project for the TokenAutoComplete project https://github.com/splitwise/TokenAutoComplete

Setup
=================
1) Clone this repo.

2) Download the latest 2.0 Jar from https://github.com/splitwise/TokenAutoComplete/releases.

3) Place the Jar in the Jars folder in the project.

4) Build Release.

5) Copy the dll from bin/Release folder into your Xamarin project. I keep mine in a lib/ folder.

6) Reference the dll in the xamarin android project.

7) I think that’s all.


Example Forms code setup
==================
For Forms:
Setup PCL side something like this:

```
namespace Quarc.Mobile.Views.CustomRenderers
{
    public class TokenField : View
    {
    }
}
```

Setup android side something like this:

```
[assembly: ExportRenderer (typeof (TokenField), typeof (TokenFieldRenderer))]
namespace Company.Mobile.iOS.Views.CustomRenderers
{
    public class TokenFieldRenderer : ViewRenderer<TokenField,PersonTokenView>
    {
        public const char MySplitChar = '§';
        public TokenFieldRenderer()
        {
        }

        protected override void OnElementChanged(ElementChangedEventArgs<TokenField> e)
        {
            base.OnElementChanged(e);
            if (e.NewElement != null)
            {
                var tokenField = new RecipientTokenView(Context);
                tokenField.SetSplitChar(MySplitChar);
                tokenField.SetBackgroundColor(Color.Transparent.ToAndroid());
                tokenField.SetPrefix("To ");
                tokenField.SetTextColor(ColorResources.DarkText.ToAndroid());
                tokenField.SetDeletionStyle(TokenAutoComplete.TokenCompleteTextView.TokenDeleteStyle.Clear);
                tokenField.SetTokenClickStyle(TokenAutoComplete.TokenCompleteTextView.TokenClickStyle.Select);
                tokenField.AllowCollapse(false);
                tokenField.PerformBestGuess(false);
                SetNativeControl(tokenField);
                tokenField.TokenAdded += TokenField_TokenAdded;
                tokenField.TokenRemoved += TokenField_TokenRemoved;
            }
        }

        void TokenField_TokenAdded (object sender, TokenAutoComplete.TokenCompleteTextView.TokenAddedEventArgs e)
        {
        }

        void TokenField_TokenRemoved(object sender, TokenAutoComplete.TokenCompleteTextView.TokenRemovedEventArgs e)
        {
        }
}
```

Also in the android side you need to setup the view with overrides:

```
public class PersonTokenView : TokenAutoComplete.TokenCompleteTextView
    {
        #region implemented abstract members of TokenCompleteTextView

        protected override Java.Lang.Object DefaultObject(string completionText)
        {
            int index = completionText.IndexOf('@');
            if (index == -1)
            {
                return new Person(completionText, completionText.replace(" ", "") + "@example.com");
            }
            else
            {
                return new Person(completionText.substring(0, index), completionText);
            }
        }

        protected override global::Android.Views.View GetViewForObject(Java.Lang.Object obj)
        {
            var l = ((Activity)Context).LayoutInflater; //(global::Android.Views.LayoutInflater)Context.GetSystemService(Activity.LAYOUT_INFLATER_SERVICE);
            LinearLayout view = (LinearLayout)l.Inflate(Company.Mobile.Android.Resource.Layout.token, (ViewGroup)Parent, false);
            view.FindViewById<TextView>(Company.Mobile.Android.Resource.Id.tokenTextView).Text = ((RecipientTokenJava)obj).ToString();

            return view;
        }
        #endregion
    }
```


```
    public class PersonJava : Java.Lang.Object
    {
        public string Name { get; set; }
        public string Email { get; set; }
        public PersonJava()
        {
        }
        public PersonJava(string name, string email)
        {
            Name = name;
            Email = email;
        }
        public override string ToString()
        {
            return Name;
        }
    }
```
