---
layout: post
title:  "Android : Spannable"
date:   2018-06-23 19:02:31 +0530
categories: Java, Android
---
A post after a long time. This time writing about Spannable. Spannable is used widely in Android. It is pretty much famous and a lot of people know about it.
Instagram app uses Spannable extensively in Status feature, where people can really change the color of what they are posting along with the size. But how to achieve such complex feats. So let's see how can we handle simple & complex text spans.

Here is how the basic Spannable looks like.
{% highlight java %}
SpannableString spanned = new SpannableString(summary); // summary is our text to display.
summarySpan.setSpan(new ForegroundColorSpan(
  ContextCompat.getColor(context, R.android.color.blue)), 0, summarySpan.length(),
    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

fancyTexView.setText(spanned);
{% endhighlight %}

The above snippet is simple. We are just creating a colored and applying it to textView. And it sets span for whole text i.e. from 0 to text.length()
Note that we can't say fancyTexView.setText(spanned.toString()); because, string will lose all it's Spannable properties. It either has to CharSequence,
SpannableString or SpannableStringBuilder.

To work with Spannable we need to take care of the flags. Which is passed as a last param for setting span.

1. SPAN_INCLUSIVE_INCLUSIVE : Includes newly appended spans to the current span if they are appended to front or end.
2. SPAN_INCLUSIVE_EXCLUSIVE : Includes newly appended spans to the current span only if they are appended at the front. If spans are appended to end, then they are considered as separate spans.
3. SPAN_EXCLUSIVE_INCLUSIVE : Includes newly appended spans to the current span only if they are appended at the end. If spans are appended to front, then they are considered as separate spans.
4. SPAN_EXCLUSIVE_EXCLUSIVE : No matter where you add the spans these are considered as separate spans.

We can even set different spans to same part of text. The snippet below will set TextView as BOLD & blue colored.
{% highlight java %}
SpannableStringBuilder builder = new SpannableStringBuilder(text);
builder.setSpan(new StyleSpan(Typeface.BOLD), 0, text.length(),
        Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
builder.setSpan(new ForegroundColorSpan(
  ContextCompat.getColor(context, R.android.color.blue)), 0, summarySpan.length(),
    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

fancyTexView.setText(builder);            
{% endhighlight %}

So far so good, but what if we want to show different Spannable in a single TextView? SetSpan will not work in this case. Meaning, if there 25 characters in a text, first 10 are blue, next 10 are red with green background & last 5 are white with black background. Yes. We can achieve such complex Spannable within same TextView, no need to have multiple TextViews.

But how do we do it? We can't
{% highlight java %}
// Make first 10 characters blue
SpannableStringBuilder builder = new SpannableStringBuilder(text);
builder.setSpan(new ForegroundColorSpan(ContextCompat.getColor(context, R.android.color.blue)), 0, 10, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

// Make next 10 characters red with green background
builder.setSpan(new ForegroundColorSpan(ContextCompat.getColor(context, R.android.color.red)), 11, 20, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
builder.setSpan(new BackgroundColorSpan(ContextCompat.getColor(context, R.android.color.green)), 11, 20, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

// Make next 5 characters white with black background
builder.setSpan(new ForegroundColorSpan(ContextCompat.getColor(context, R.android.color.white)), 21, 24, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
builder.setSpan(new BackgroundColorSpan(ContextCompat.getColor(context, R.android.color.black)), 21, 24, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

fancyTexView.setText(builder);   
{% endhighlight %}

Seems fun & easy. Can we have custom behaviour for out spans? Like rounded corners, drawing? Yes. We can extend ReplacementSpan & control our drawing. There are other helper classes which we can extend like ImageSpan, AudioSpan, StyleSpan etc.

Note that we are setting spannable for out edit text & we are typing something, we need ReplacementSpan. Because we in Android <Space> is rendered in a different way than other characters.
Let's see example of ReplacementSpan. The below example shows how we can add margin, padding, corners to spannable.

{% highlight java %}
public class MyBackgroundSpan extends ReplacementSpan  {

    @ColorInt private int mBackgroundColor;
    @ColorInt private int mForegroundColor;
    private RectF mRect;
    private int mCornerRadius;
    private float mPaddingLeft;
    private float mPaddingRight;
    private float mMarginLeft;
    private float mMarginRight;

    public MyBackgroundSpan(@ColorInt int backgroundColor, @ColorInt int foregroundColor,
            int paddingLeft, int paddingRight, int marginLeft, int marginRight, int radius) {
        super();
        mBackgroundColor = backgroundColor;
        mForegroundColor = foregroundColor;
        mRect = new RectF();
        mPaddingRight = paddingRight;
        mPaddingLeft = paddingLeft;
        mMarginLeft = marginLeft;
        mMarginRight = marginRight;
        mCornerRadius = radius;
    }

    @Override
    public int getSize(@NonNull Paint paint, CharSequence text, int start, int end,
            @Nullable Paint.FontMetricsInt fm) {
        return (int) (mMarginLeft + mPaddingRight + paint.measureText(text.subSequence(start, end)
                .toString()) + mPaddingLeft);
    }

    @Override
    public void draw(@NonNull Canvas canvas, CharSequence text, int start, int end, float x,
            int top, int y, int bottom, @NonNull Paint paint) {

        // draw background color
        float width = paint.measureText(text.subSequence(start, end).toString());
        paint.setColor(mBackgroundColor);
        mRect.set(x + mMarginLeft, top, x + width + mMarginLeft + mPaddingLeft + mPaddingRight,
                bottom);
        canvas.drawRoundRect(mRect, mCornerRadius, mCornerRadius, paint);

        // draw text
        paint.setColor(mForegroundColor);
        canvas.drawText(text, start, end, x + mMarginRight + mPaddingLeft,
                y - paint.getFontMetricsInt().descent / 2, paint);
    }
}
{% endhighlight %}

It is fun to play around spannable. There are a lot of things we can do with them achieving great UX. But we should avoid using them a lot as their drawing can get costly sometimes, especially ReplacementSpan.
