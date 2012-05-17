I've added a second "Lesson" to the Tutorial app, and also done a little refactoring to make the Tutorial a bit more readable.

This new "Lesson" is called PullToRefresh, and demonstrates a view on each platform where the user can "Pull down" the list to get more data.

![PullToRefresh](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/PullToRefresh.png)

## PullToRefreshViewModel

The ViewModel in this example is https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.Core/ViewModels/Lessons/PullToRefreshViewModel.cs

It implements a ViewModel which presents a public API of:

```
        public ObservableCollection<SimpleEmail> Emails { get; set; }
        public IMvxCommand RefreshHeadCommand { get; }
        public IMvxCommand RefreshTailCommand { get; }
        public bool IsRefreshingHead { get; }
        public bool IsRefreshingTail { get; }
```

The idea of this is simple enough - it allows the View to show the list of Email objects, and to request more items from either the top (Head) or bottom (Tail) using the IMvxCommand's



## WP7

To implement the Pull down effect in WP7, we use a technique from http://blogs.msdn.com/b/slmperf/archive/2011/06/30/windows-phone-mango-change-listbox-how-to-detect-compression-end-of-scroll-states.aspx

This is wrapped up in a platform specific user control - https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.WindowsPhone/Controls/ExtendedListBox.xaml - which contains some fairly complex WP7 code (not covered here - see the earlier blog link, or consider an alternative at http://blogs.msdn.com/b/jasongin/archive/2011/04/13/pull-down-to-refresh-a-wp7-listbox-or-scrollviewer.aspx).

The View code is then very simple databinding:

```
            <ProgressBar 
                Grid.Row="0"
                IsIndeterminate="True"
                Visibility="{Binding IsRefreshingHead, Converter={StaticResource MvxVisibilityConverter}}"
                Width="400"
                VerticalAlignment="Top"
                />
            <StackPanel Orientation="Horizontal"
                        Grid.Row="0">
                <TextBlock Text="Number of emails shown: "></TextBlock>
                <TextBlock Text="{Binding Emails.Count}"></TextBlock>
            </StackPanel>

            <Controls:ExtendedListBox 
                Grid.Row="1"
                ItemsSource="{Binding Emails}"
                LoadHead="{Binding RefreshHeadCommand}"
                LoadTail="{Binding RefreshTailCommand}"
                >
                <Controls:ExtendedListBox.ItemTemplate>
                    <DataTemplate>
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="Auto"></ColumnDefinition>
                                <ColumnDefinition Width="*"></ColumnDefinition>
                            </Grid.ColumnDefinitions>
                            <TextBlock Text="{Binding From}" Margin="12"/>
                            <StackPanel Grid.Column="1" Margin="12">
                                <TextBlock Text="{Binding Header}"/>
                                <TextBlock Text="{Binding Message}"/>
                            </StackPanel>
                        </Grid>
                    </DataTemplate>
                </Controls:ExtendedListBox.ItemTemplate>
            </Controls:ExtendedListBox>
            
            <!-- warning - in production code use the high performance Indeterminate Progress Bar -->
            <ProgressBar 
                Grid.Row="1"
                IsIndeterminate="True"
                Visibility="{Binding IsRefreshingTail, Converter={StaticResource MvxVisibilityConverter}}"
                Width="400"
                VerticalAlignment="Bottom"
                />
```

## Android

To implement the Pull down effect in Android, we use a technique from https://github.com/guillep/PullToRefresh (used under MIT license).

This control has been ported to C# in https://github.com/slodge/MvvmCross/tree/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.Droid/Controls/PullToRefresh

We've then inherited from that very Java-esque control in order to provide an IMvxCommand-compatible API - see  https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.Droid/Controls/BindingPullToRefreshListView.cs

All of this is, of course, quite technical and platform specific - which is exactly why it's in the Android UI project.

With that control in place, everything now is done by binding - especially in the page file - https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.Droid/Resources/Layout/Page_PullToRefreshView.axml:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical">
  <LinearLayout
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:orientation="horizontal">
    <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Number of emails shown: "
    />
    <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      local:MvxBind="{'Text':{'Path':'Emails.Count'}}"
    />
  </LinearLayout>
  <tutorial.ui.droid.controls.BindingPullToRefreshListView
      android:layout_width="fill_parent"
      android:layout_height="fill_parent"
      local:MvxBind="{'PullDownRefreshRequested':{'Path':'RefreshHeadCommand'},'PullUpRefreshRequested':{'Path':'RefreshTailCommand'},'IsRefreshingHead':{'Path':'IsRefreshingHead'},'IsRefreshingTail':{'Path':'IsRefreshingTail'}}"
  />
</LinearLayout>
```

although there is also a little binding in the the list layout too - in https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.Droid/Resources/Layout/pull_to_refresh_bindable_list.axml :

```
    <Mvx.MvxBindableListView
        android:id="@android:id/list"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        local:MvxBind="{'ItemsSource':{'Path':'Emails'},'ItemClick':{'Path':'ShowItemCommand'}}"
        local:MvxItemTemplate="@layout/listitem_email"
    />
```

and that's it - the Activity itself is very empty:

```
    [Activity]
    public class PullToRefreshView
        : MvxBindingActivityView<PullToRefreshViewModel>
    {
        protected override void OnViewModelSet()
        {
            SetContentView(Resource.Layout.Page_PullToRefreshView);
        }
    }
```


## iPhone

To implement the Pull down effect in iPhone, we use a technique from https://github.com/sichy/TableViewPullToRefreshFolding (used under what @sichy said is a "public" license).

This control has been imported into https://github.com/slodge/MvvmCross/blob/master/Sample%20-%20Tutorial/Tutorial/Tutorial.UI.Touch/Controls/FoldingTableViewController.cs and we've just modified it slightly in order to provide and use the IMvxCommand.

As with Droid and WP7 the details of the PullToRefresh control don't really matter - the point of MvvmCross is that we can use these complicated Native controls - this project itself isn't about how to create them.

For the View, we use a XIB file (from the interface designer) to generate most of the UI, then create and bind the table using the code:

```
        public PullToRefreshView (MvxShowViewModelRequest request) 
            : base (request, "PullToRefreshView", null)
        {
        }
        
        public override void ViewDidLoad ()
        {
            base.ViewDidLoad ();
            
            var foldingTvc = new FoldingTableViewController(TableViewHolder.Frame, UITableViewStyle.Grouped);

            var tableSource = new TableViewDataSource(foldingTvc.TableView);

            this.AddBindings(
                new Dictionary<object, string>()
                    {
                        { tableSource, "{'ItemsSource':{'Path':'Emails'}}" },
                        { foldingTvc, "{'RefreshHeadCommand':{'Path':'RefreshHeadCommand'},'Refreshing':{'Path':'IsRefreshingHead'}}" },
                        { NumberOfEmailsLabel, "{'Text':{'Path':'Emails.Count'}}" }
                    });
            
            foldingTvc.TableView.Source = tableSource;
            foldingTvc.TableView.ReloadData();
            
            Add(foldingTvc.View);			
        }
```

Note: the iPhone sample now uses a custom cell - driven by a Nib file. The loading of that cell is a bit complicated - but that's all abstracted away inside the cell class, and it's complicated because of Obj-C, Cocoa, and MonoTouch, not because of Mvvm - so I won't cover that more here (for more info, see www.alexyork.net/blog/2011/07/18/creating-custom-uitableviewcells-with-monotouch-the-correct-way/ - including the comments which cover some xcode 4 updates).