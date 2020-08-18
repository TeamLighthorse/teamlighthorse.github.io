---
layout: post
title: Testing Material Dialogs That Emit Data
tags: [angular, testing]
excerpt_separator: <!--more-->
author: docHologram
---

## What's the story, morning glory?

Unit testing involving a component's complimentary dialog component can be tricky, especially when we are expecting the component to react to an emission from that dialog. While such testing is technically, integration testing, it is no less important considering that so much of Lighthouse involves the use of dialogs.
<!--more-->

## Let's go off on a tangent first

When we first began prototyping the overall layout and design for Lighthouse, way back yonder in 2017, we decided to sew the Angular Material library directly into the lifeblood of the design. We were confident making that choice since the Angular Material team is also a part of the Angular team, so we can rest assured that the Material library will always be kept up-to-date alongside subsequent Angular versions. One of our most prolific uses from the Material library are dialogs. This article will focus on a common pattern we have in Lighthouse that involves pop-up Material dialogs that emit data for the parent component to consume and how we can test that data exchange.

In Lighthouse, we have a page-level component called Shipment Spotlight that provides the details of any shipment in our system. If the shipment we're shining a spotlight on has not been dispatched yet, then most of that shipment's data can be updated. We present these in the form of microedits, such as editing the contact on the destination location or adding a commodity.

![Edit Commodity](/assets/img/update-commodity.gif)

The result of these microedits gets emitted from the dialog when we hit the save button and the dialog then closes. The parent component--in this case, the row of commodity tiles--then captures this data in order to pass it up the chain of command to the Spotlight component.

We want to test two functional ideas here:
1. does the dialog open and pass in the necessary data required
2. when the editing is complete and the dialog emits an event stating such, does the parent component pick up on that event emission.

OK, now we're back. I guess it wasn't so tangential after all.

Next, let's look at the template and component functions...

## Mother component and the child dialog

_Disclaimer: in the world of Lighthouse, the terms "line item" and "commodity" are synonymous. While the team has decided that "commodity" will be the ubiquitous term going forward, references to "line items" are still littered throughout the code._

Our template contains an *ngFor directive that loops through the line items and displays each one as a tile.

```
<div class="lh-tile-row">
	<div class="card lh-tile"
		*ngFor="let lineItem of lineItems; index as i; count as numItems">
		<div class="lh-tile__header clearfix"
			*ngIf="(numItems > 1) && isEditable">
			<button mat-icon-button
				class="float-right">
				<mat-icon class="lh-tile__icon"
					(click)="openDeleteLineItemsDialog(lineItem)">
					clear
				</mat-icon>
			</button>
		</div>
		<div class="lh-tile__content"
			(click)="openEditLineItemsDialog(lineItem)">
			<ul class="lh-tile__description">
				<li class="lh-tile__title">
					{{ lineItem.description }}
				</li>
				<li>
					Class {{ lineItem.class }}
					<mat-icon class="lh-tile__icon"
						*ngIf="lineItem.isHazardous"
						color="accent">
						warning
					</mat-icon>
				</li>
				<li>{{ lineItem.weight }} lbs, {{ lineItem.handlingUnits }} H/U</li>
				<li [innerHTML]="displayDimensions(lineItem.dimensions)"></li>
			</ul>
		</div>
	</div>
</div>
```

If we look closely at `div.lh-tile__content`, which is the content inside the tile, we see a click event that is handled by `openEditLineItemsDialog(lineItem)` that passes in the line item to edit.

In the component class:

```
openEditLineItemsDialog(lineItem?: LineItem): void {
    if (!this.isEditable) { return; }

    // If no commodity gets passed in, then the user is adding a commodity
    const editLineItemsInfo: EditLineItemsInfo = {
        title: !lineItem ? 'Add Commodity' : 'Edit Commodity',
        lineItem,
        shipmentIdentifier: this.lineItemsInfo.shipmentIdentifier,
        shipmentStatus: this.lineItemsInfo.shipmentStatus
    };
    this.editLineItemsDialogRef = this.dialog.open(EditLineItemsDialogComponent, {
        width: '800px',
        maxHeight: '100vh',
        data: editLineItemsInfo
    });

    const editSuccess = this.editLineItemsDialogRef.componentInstance.dialogEditSuccess
        .subscribe(_ => this.editLineItemsSuccess.emit());
    this.subscriptions.push(editSuccess);

    // ...some other stuff
}
```

You'll notice that passing in a `lineItem` is optional. This would be the case if the user is "adding" a commodity as opposed to "editing" a commodity. We will stay focused on editing a commodity for purposes of this post.

What is essentially going on here is:
1. `editLineItemsInfo` is preparing the data to send into the dialog
2. the dialog opens and passes in `editLineItemsInfo` and returns a reference to that dialog, `this.editLineItemsDialogRef`
3. a subscription is established that will listen for `dialogEditSuccess` emissions from `this.editLineItemsDialogRef`

## Time to Test!

First let's test that the opening of the dialog actually occurs and passes in the data expected. In order to test this, let's create a spy for the dialog's `open` method and check to see if the spy gets called with the appropriate data.

In the following, `testLineItemsInfo` is some test data we created and imported during the setup of the spec file.

```
it('should open the dialog and pass in the line item to edit', () => {
    const testLineItem = testLineItemsInfo.lineItems[0];

    const dialogOpenSpy = spyOn(component.dialog, 'open').and.callThrough();

    const expectedAddOptions = {
        width: '800px',
        maxHeight: '100vh',
        data: {
            title: 'Edit Commodity',
            lineItem: testLineItem,
            shipmentIdentifier: component.lineItemsInfo.shipmentIdentifier,
            shipmentStatus: component.lineItemsInfo.shipmentStatus
        }
    };

    component.openEditLineItemsDialog(testLineItem);
    expect(dialogOpenSpy).toHaveBeenCalledWith(EditLineItemsDialogComponent, expectedAddOptions);
});
```

Next, we will test that we are properly reacting to the dialog's `dialogEditSuccess` emission by manually call it from within the test:

```
it('should emit the editLineItemsSuccess event when complete', () => {
    // spy on the event emission that the parent should call after it receives the dialog's event emission
    const editSuccessSpy = spyOn(component.editLineItemsSuccess, 'emit');

    component.openEditLineItemsDialog();
    // manually emit the event from the dialog
    component.editLineItemsDialogRef.componentInstance.dialogEditSuccess.emit();

    expect(editSuccessSpy).toHaveBeenCalled();
});
```

You could also test for a dialog emission that contains data by adding that as an argument to `dialogEditSuccess.emit()`, such as in the following:

```
// component class subscription
const editSuccess = this.editLineItemsDialogRef.componentInstance.dialogEditSuccess
    .subscribe((value: number) => this.editLineItemsSuccess.emit(value));
```

...and then in the spec file:

```
it('should emit the editLineItemsSuccess event when complete', () => {
    const editSuccessSpy = spyOn(component.editLineItemsSuccess, 'emit');

    component.openEditLineItemsDialog();
    component.editLineItemsDialogRef.componentInstance.dialogEditSuccess.emit(1000000);

    expect(editSuccessSpy).toHaveBeenCalledWith(1000000);
});
```

Ta-da! That be it! Now go off and add unit tests for all the dialog emissions!