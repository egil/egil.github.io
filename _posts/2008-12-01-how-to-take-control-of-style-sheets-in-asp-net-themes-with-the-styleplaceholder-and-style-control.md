---
layout: post
title: How to take control of style sheets in ASP.NET Themes with the StylePlaceholder
  and Style control
created: 1228105462
redirect_from:
  - /2008/11/30/how-take-control-style-sheets-aspnet-themes-styleplaceholder-and-style-control/
---
The problem is pretty simple. When using ASP.NET Themes you do not have much say in how your style sheets are rendered to the page.

The render engine adds all the style sheets you have in your themes folder in alphabetic order, using the `<link href="..."` notation.

<!--break-->

We all know the order of the style sheets are important, luckily asp.net's shortcomings can be circumvented by prefixing the style sheets with 01, 02, ... , 99, and thus forcing the order you want (see [Rusty Swayne blog post](http://rustyswayne.com/post.aspx?id=cba116ea-b672-4c90-9f4e-18b70ca2f50a) on the technique for more information).

This is especially important if you use a reset style sheet, which I highly recommend; it makes it much easier to style a site in a consistent form across browsers (take a look at [Reset Reloaded from Eric Meyer](http://meyerweb.com/eric/thoughts/2007/05/01/reset-reloaded/)).

You also miss out of the possibility to specify a media type (e.g. screen, print, projection, braille, speech). And if you prefer to include style sheets using the @import method, you are also left out in the cold.

Another missing option is Conditional Comment, which is especially useful if you use an "ie-fix.css" style sheet.

Before I explain how the StylePlaceHolder and Style control resolve the above issues, credit where credit is due, my solution is inspired by [Per Zimmermann's blog post](http://www.dentaku.com/2007/01/take-control-over-stylesheet-order-and-media-when-using-asp-net-2-0-themes.aspx) on the subject.

The StylePlaceHolder control is placed in the header section of your master page or page. It can host one or more Style controls, and will remove styles added by the render engine by default, and add its own (it will only remove styles added from the current active theme).

The Style control can both host inline styles in-between its opening and closing tags and a reference to a external style sheet file through its CssUrl property. Through other properties you control how the style sheet it renders to the page.

Let me show an example. Consider a simple web site project with a master page and a theme with three style sheets: 01reset.css, 02style.css, 99iefix.cs. Note: I have named them using prefixing technique described earlier, as it makes for a better design time inside Visual Studio experience. Also, the tag prefix of the custom controls is "ass:"

In the master page's header section, add:

~~~
<ass:StylePlaceHolder ID="StylePlaceHolder1" runat="server" SkinID="ThemeStyles" />
~~~

In your theme directory, add a skin file (e.g. Styles.skin) and add the following content:

~~~
<ass:StylePlaceHolder runat="server" SkinId="ThemeStyles">
    <ass:Style CssUrl="~/App_Themes/Default/01reset.css" />
    <ass:Style CssUrl="~/App_Themes/Default/02style.css" />
    <ass:Style CssUrl="~/App_Themes/Default/99iefix.css" ConditionCommentExpression="[if IE]" />
</ass:StylePlaceHolder>
~~~

That is basically it. There are a more properties on the Style control that can be used to control the rendering, but this is the basic setup. With that in place, you can easily add another theme and replace all the styles, since you only need to include a different skin file.

The following is the code for the Style control:

~~~csharp
using System.ComponentModel;
using System.Security.Permissions;
using System.Web;
using System.Web.UI;

[assembly: TagPrefix("Assimilated.Extensions.Web.Controls", "ass")]
namespace Assimilated.WebControls.Stylesheet
{
    [AspNetHostingPermission(SecurityAction.Demand, Level = AspNetHostingPermissionLevel.Minimal)]
    [AspNetHostingPermission(SecurityAction.InheritanceDemand, Level = AspNetHostingPermissionLevel.Minimal)]
    [DefaultProperty("CssUrl")]
    [ParseChildren(true, "InlineStyle")]
    [PersistChildren(false)]
    [ToolboxData("<{0}:Style runat=\"server\"></{0}:Style>")]
    [Themeable(true)]
    public class Style : Control
    {
        public Style()
        {
            // set default value... for some reason the DefaultValue attribute do
            // not set this as I would have expected.
            TargetMedia = "All";
        }

        #region Properties

        [Browsable(true)]
        [Category("Style sheet")]
        [DefaultValue("")]
        [Description("The url to the style sheet.")]
        [UrlProperty("*.css")]
        public string CssUrl
        {
            get; set;
        }

        [Browsable(true)]
        [Category("Style sheet")]
        [DefaultValue("All")]
        [Description("The target media(s) of the style sheet. See http://www.w3.org/TR/REC-CSS2/media.html for more information.")]
        public string TargetMedia
        {
            get; set;
        }

        [Browsable(true)]
        [Category("Style sheet")]
        [DefaultValue(EmbedType.Link)]
        [Description("Specify how to embed the style sheet on the page.")]
        public EmbedType Type
        {
            get; set;
        }

        [Browsable(false)]
        [PersistenceMode(PersistenceMode.InnerDefaultProperty)]
        public string InlineStyle
        {
            get; set;
        }

        [Browsable(true)]
        [Category("Conditional comment")]
        [DefaultValue("")]
        [Description("Specifies a conditional comment expression to wrap the style sheet in. See http://msdn.microsoft.com/en-us/library/ms537512.aspx")]
        public string ConditionalCommentExpression
        {
            get; set;
        }

        [Browsable(true)]
        [Category("Conditional comment")]
        [DefaultValue(CommentType.DownlevelHidden)]
        [Description("Whether to reveal the conditional comment expression to downlevel browsers. Default is to hide. See http://msdn.microsoft.com/en-us/library/ms537512.aspx")]
        public CommentType ConditionalCommentType
        {
            get; set;
        }

        [Browsable(true)]
        [Category("Behavior")]
        public override string SkinID { get; set; }

        #endregion

        protected override void Render(HtmlTextWriter writer)
        {            
            // add empty line to make output pretty
            writer.WriteLine();

            // prints out begin condition comment tag
            if (!string.IsNullOrEmpty(ConditionalCommentExpression))
                writer.WriteLine(ConditionalCommentType == CommentType.DownlevelRevealed ? "<!{0}>" : "<!--{0}>",
                                 ConditionalCommentExpression);

            if (!string.IsNullOrEmpty(CssUrl))
            {               
                // add shared attribute
                writer.AddAttribute(HtmlTextWriterAttribute.Type, "text/css");

                // render either import or link tag
                if (Type == EmbedType.Link)
                {
                    // <link href=\"{0}\" type=\"text/css\" rel=\"stylesheet\" media=\"{1}\" />
                    writer.AddAttribute(HtmlTextWriterAttribute.Href, ResolveUrl(CssUrl));
                    writer.AddAttribute(HtmlTextWriterAttribute.Rel, "stylesheet");
                    writer.AddAttribute("media", TargetMedia);
                    writer.RenderBeginTag(HtmlTextWriterTag.Link);
                    writer.RenderEndTag();
                }
                else
                {
                    // <style type="text/css">@import "modern.css" screen;</style>
                    writer.RenderBeginTag(HtmlTextWriterTag.Style);
                    writer.Write("@import \"{0}\" {1};", ResolveUrl(CssUrl), TargetMedia);
                    writer.RenderEndTag();
                }
            }

            if(!string.IsNullOrEmpty(InlineStyle))
            {
                // <style type="text/css">... inline style ... </style>
                writer.AddAttribute(HtmlTextWriterAttribute.Type, "text/css");
                writer.RenderBeginTag(HtmlTextWriterTag.Style);
                writer.Write(InlineStyle);
                writer.RenderEndTag();
            }

            // prints out end condition comment tag
            if (!string.IsNullOrEmpty(ConditionalCommentExpression))
            {
                // add empty line to make output pretty
                writer.WriteLine();
                writer.WriteLine(ConditionalCommentType == CommentType.DownlevelRevealed ? "<![endif]>" : "<![endif]-->");
            }
        }
    }
    
    public enum EmbedType
    {        
        Link = 0,
        Import = 1,
    }

    public enum CommentType
    {
        DownlevelHidden = 0,
        DownlevelRevealed = 1
    }
}
~~~

And the code for the StylePlaceHolder control:

~~~
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Security.Permissions;
using System.Web;
using System.Web.UI;
using System.Web.UI.HtmlControls;

[assembly: TagPrefix("Assimilated.Extensions.Web.Controls", "ass")]
namespace Assimilated.WebControls.Stylesheet
{
    [AspNetHostingPermission(SecurityAction.Demand, Level = AspNetHostingPermissionLevel.Minimal)]
    [AspNetHostingPermission(SecurityAction.InheritanceDemand, Level = AspNetHostingPermissionLevel.Minimal)]
    [DefaultProperty("SkinID")]
    [ToolboxData("<{0}:StylePlaceHolder runat=\"server\" SkinID=\"ThemeStyles\"></{0}:StylePlaceHolder>")]
    [ParseChildren(true, "Styles")]
    [Themeable(true)]
    [PersistChildren(false)]
    public class StylePlaceHolder : Control
    {
        private List<Style> _styles;

        [Browsable(true)]
        [Category("Behavior")]
        [DefaultValue("ThemeStyles")]
        public override string SkinID { get; set; }

        [Browsable(false)]
        public List<Style> Styles
        {
            get
            {
                if (_styles == null)
                    _styles = new List<Style>();
                return _styles;
            }
        }

        protected override void CreateChildControls()
        {
            if (_styles == null)
                return;

            // add child controls
            Styles.ForEach(Controls.Add);
        }

        protected override void OnLoad(EventArgs e)
        {
            base.OnLoad(e);

            // get notified when page has finished its load stage
            Page.LoadComplete += Page_LoadComplete;
        }

        void Page_LoadComplete(object sender, EventArgs e)
        {
            // only remove if the page is actually using themes
            if (!string.IsNullOrEmpty(Page.StyleSheetTheme) || !string.IsNullOrEmpty(Page.Theme))
            {
                // Make sure only to remove style sheets from the added by
                // the runtime form the current theme.
                var themePath = string.Format("~/App_Themes/{0}",
                                              !string.IsNullOrEmpty(Page.StyleSheetTheme)
                                                  ? Page.StyleSheetTheme
                                                  : Page.Theme);

                // find all existing stylesheets in header
                var removeCandidate = Page.Header.Controls.OfType<HtmlLink>()
                    .Where(link => link.Href.StartsWith(themePath)).ToList();

                // remove the automatically added style sheets
                removeCandidate.ForEach(Page.Header.Controls.Remove);
            }
        }

        protected override void AddParsedSubObject(object obj)
        {
            // only add Style controls
            if (obj is Style)
                base.AddParsedSubObject(obj);
        }

    }
}
~~~

So what do you guys think? Is this a good solution to the asp.net theme problem? And what about the code? These two controls are my first attempt at writing custom server controls, so any input is very much welcome.

Get the code below, including Visual Studio 2008 solution and .dll if you just want to use the controls.
