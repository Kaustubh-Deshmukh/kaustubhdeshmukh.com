---
layout: post
title:  "Android : Drag & drop with ChipsView and AutoComplete"
date:   2017-11-25 08:52:50 +0530
categories: Android
---
I will be talking about how I achieved Drag & drop between the AutoComplete ChipsView. We always loved the way Gmail's ChipsView worked while entering email addresses. They have one of the best UX but for us at [Astro](https://helloastro.com/), we wanted to extend functionality to include drag & drop. Much better version.

Brace yourself, a long post coming.
To design something which will never go beyond the bounds of device you must have either view's width set to `MATCH_PARENT` or need self adjusting layout. I decided to go with later approach. With [FlowLayout](https://github.com/ApmeM/android-flowlayout)

Bang! My first problem is solved. Second problem was I wanted email addresses to look like [ChipsView](https://material.io/guidelines/components/chips.html). Simple enough. Just followed Google's guidelines.

Now, next problem a big one, drag and drop. As we are an email company, we have to support to, cc & bcc fields. I can not just simply add same code for every view. Hence decided to write a single [DragListener](https://developer.android.com/reference/android/view/View.OnDragListener.html).

Whenever I add a new ChipsView to my flow layout, I would add the same instance of OnDragListener & OnLongClickListener.
Remember, it is always good to start dragging only on long click.

{% highlight java %}
chipView.setOnLongClickListener(new OnLongClickListener() {
    @Override
    public boolean onLongClick(View view) {
        // TODO.. add your any validation stuff here..
        ClipData data = ClipData.newPlainText(String.valueOf(view.getId()), "");
        View.DragShadowBuilder shadowBuilder = new View.DragShadowBuilder(
                view);
        view.startDrag(data, shadowBuilder, view, 0); // Start dragging of view
        FlowLayout flowLayout = (FlowLayout) view.getParent();
        view.setVisibility(GONE); // This will make out source view disappear from current viewgroup
        flowLayout.invalidate();
        return true; // We consumed it.
    }
});
// Our ChipsView need to know drag events.
chipView.setOnDragListener(dragListener);
{% endhighlight %}

We need to set OnDragListener to all the views `on which we are planning to drop source view`. onDrag(...) event will eventually give us targetView and DragEvent. DragEvent will contain the source view. Hence we will add OnDragListener to every ChipView.

{% highlight java %}
final View sourceView = (View) dragEvent.getLocalState();
// above line will give us source view.
{% endhighlight %}

{% highlight java %}
public class AstroDragListener implements View.OnDragListener {
  @Override
  public boolean onDrag(View targetView, DragEvent event) {
    // here targetView is view on which we have dropped our source view
  }
}
{% endhighlight %}

There are multiple states of DragEvent. `DragEvent.ACTION_DRAG_ENTERED, DragEvent.ACTION_DRAG_ENDED, DragEvent.ACTION_DROP`. Most important is DragEvent.ACTION_DROP. Which tells us when view is dropped on the other view. We need to add our logic, how we need to show dropped view on the target ViewGroup.

What I did was I calculated the position of view on which the SourceView is dropped & added view in front of view.
{% highlight java %}
// We have already populated sourceView above
final FlowLayout sourceContainer = (FlowLayout) sourceView.getParent();
final FlowLayout targetContainer = (AstroFlowLayout) targetView;

int sourcePosition = ViewUtil.getViewPositionInParent(sourceContainer, sourceView);
int targetPosition = ViewUtil.getViewPositionInParent(targetContainer, targetView);

if (event.getResult()) {
  // As we had set visibility of source view to GONE, its time to make it visible
  // just for safety
  sourceView.setVisibility(View.VISIBLE);
  // As drop action is complete, remove source view from source target container
  sourceContainer.removeChildView(sourceView);

  // And add source view to target container
  targetContainer.addViewAt(targetPosition, sourceView);
}
{% endhighlight %}

There are a few validations I made for edge cases, but this version should work for everyone.
