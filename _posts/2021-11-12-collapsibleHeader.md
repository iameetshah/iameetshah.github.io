---
title: Collapsible Header with Search in SwiftUI
author: Meet Shah
date: 2021-11-12
category: Jekyll
layout: post
---

I have been searching for a tutorial in SwiftUI which explains how to make an collapsible header 
which also allows search within sub header and after going through many articles, 
finally managed to understand and got it working. So I decided to write an article 
which will be informative and helpful to someone new to SwiftUI.

Final Result :

![screenshot1](../../../../assets/collapsibleList.png){: width="280" } ![screenshot2](../../../../assets/collapsibleListSearch.png){: width="280" }

Let’s get started by creating a struct for `HeaderItem` which conforms to `Identifiable` protocol

```swift
struct HeaderItem :Identifiable {
let id:Int
let name:String // To be tapped for expanding 
let details:String // as per your structure -- not needed
let subHeader:[SubHeaderItem]// One or more items displayed on collapse
}
```

Moving to next step by creating struct for `SubHeaderItem` i.e. items to be displayed on collapse (i.e.tapping the header) :

```swift
struct SubHeaderItem:Identifiable {
let id: Int
let name: String // To be displayed on collapse
let otherPropery: String // as per your structure -- not needed
}
```

Now, since both HeaderItem and SubHeaderItem are created we can start with creating a View in SwiftUI,
let’s name it `MenuView` :

```swift
struct MenuView: View {
var headerItem:[HeaderItem]!
   var body: some View {
    scrollForEach
   }
   var scrollForEach: some View {
    ScrollView { 
     ForEach(headerItem) { header in
      Text("\(header.name)")
     }
}
```

In above code, We have used a ScrollView with ForEach to loop through each of the `HeaderItem` and display each header name using Text View. 
Pretty simple, right?

Before we create `SubMenuView`, we need to first understand how to differentiate between selection of items i.e. 
how can we provide user an option to select header or one of the items from Sub Header. So we will create a 
Text View with “Select all” to indicate that header(in other words, all items belonging to that header) are selected 
and Text View with sub header name to indicate the respective `SubHeaderItem` is selected. 
Next, we also need to add an arrow next to the header name which will expand /collapse on user’s tap. 
Now that we have a clear understanding of what needs to be done we can start creating the `SubMenuView` :

```swift
struct SubMenuView: View {
    let header: HeaderItem
    @Binding var selectedItem:SubHeaderItem?
    @Binding var selectedHeader:HeaderItem?
    @State var isExpanded: Bool
    let searchText: String
    var body: some View {
        HStack {
            content
            Spacer()
        }
        .contentShape(Rectangle())
    }
    private var content: some View {
        VStack(alignment: .leading) {
            HStack {
                Text(header.name)
                    .font(.system(size: 16))
                    .padding(EdgeInsets(top: 0, leading: 20, bottom: 0, trailing: 0))
                Spacer()
                Image("arrow_expand").rotationEffect(.degrees(isExpanded ? 90 : 270))
            }
            .contentShape(Rectangle())
            .onTapGesture {
                isExpanded.toggle() // To expand/collapse based on user's tap
            }
            // On Expanding display "Select all" and all Sub Header Items
            if isExpanded {
                // To display Select all and update selection on tap
                VStack(alignment: .leading) {
                    Divider()
                    HStack {
                        Text("Select all")
                            .font(.system(size: 16))
                        Spacer()
                        Image(selectedHeader == header ? "RadioChecked" : "RadioUnchecked")
                    }
                    .contentShape(Rectangle())
                    .onTapGesture {
                        selectedItem = nil
                        selectedHeader = header
                    }
                    // To display each of SubHeader Items and update selection on tap
                    ForEach(header.subHeader) { subHeader in
                        Divider()
                        HStack {
                            Text(subHeader.name)
                                .font(.system(size: 16))
                            Spacer()
                            Image(selectedItem == subHeader ? "RadioChecked" : "RadioUnchecked")
                        }
                        .contentShape(Rectangle())
                        .onTapGesture {
                            selectedItem = subHeader
                            selectedHeader = nil
                        }
                    }
                }
            }
        }
    }
}
```

As you might have noticed in above snippet there is a variable `“searchText”`. Yes, thats right now we move to next step i.e. Search Functionality.
As always we need to take a step back and first understand what needs to be done to implement search functionality of Sub header items. 
First and foremost, we need a create a view for `SearchBar`. So let’s create that :

```swift
struct SearchBar: View {
    @Binding var text: String
    @Binding var isEditing:Bool
    var body: some View {
        HStack {
            TextField("Search in sub header eg.'5' shows header 2 ...", text: $text)
                .background(Color.gray)
                .cornerRadius(8)
                .overlay(
                    HStack {
                        Image(systemName: "magnifyingglass")
                            .foregroundColor(.gray)
                            .frame(minWidth: 0, maxWidth: .infinity, alignment: .leading)
                            .padding(.leading, 8)
                        if isEditing {
                            Button(action: {
                                self.text = ""
                            }) {
                                Image(systemName: "multiply.circle.fill")
                                    .foregroundColor(.gray)
                                    .padding(.trailing, 8)
                            }
                        }
                    })
                .padding(.horizontal, 10)
                .onTapGesture {
                    self.isEditing = true
                }
            if isEditing {
                Button(action: {
                    self.isEditing = false
                    self.text = ""
                    // Dismiss the keyboard
                    UIApplication.shared.sendAction(#selector(UIResponder.resignFirstResponder), to: nil, from: nil, for: nil)
                }) {
                    Text("Cancel").font(.system(size: 16)).foregroundColor(Color.primary)
                }
                .padding(.trailing, 10)
                .transition(.move(edge: .trailing))
                .animation(.default)
            }
        }
    }
}
```

Now we need to go back to MenuItem View and add above SearchBar in its View hierarchy. Also we need to add 
a menuFilter function which will filter `HeaderItem` based on searchText so that only those `HeaderItem's` are displayed 
wherein `SubHeaderItem` contains the searchText.

```swift
struct MenuView: View {
    var dismiss: (() -> Void)?
    @State private var searchText = ""
    var headerItem:[HeaderItem] = MenuItems.sample
    @State var isEditing:Bool = false
    @State var selectedItem:SubHeaderItem?
    @State var selectedHeader:HeaderItem?
    var body: some View {
        searchableView
        scrollForEach
    }
    var scrollForEach: some View {
        ScrollView {
            ForEach(searchText.isEmpty ? headerItem : menuFilter()) { header in
                SubMenuView(header: header,selectedItem: $selectedItem, selectedHeader: $selectedHeader,isExpanded: false, searchText: searchText)
                    .modifier(ListRowModifier())
                    .animation(.linear(duration: 0.3))
            }
        }
    }
    var searchableView: some View {
        VStack(alignment: .leading) {
            SearchBar(text: $searchText, isEditing: $isEditing)
                .padding(EdgeInsets(top: 0, leading: 10, bottom: 0, trailing: 10))
            Text("Display data for")
                .font(.system(size: 16)).foregroundColor(Color.gray)
            Divider()
        }
    }
    // To filter only those HeaderItems wherein SubHeaderItem contains the searchText
    private func menuFilter() -> [HeaderItem] {
        return headerItem.filter { (header) -> Bool in
            for subHeader in header.subHeader {
                if subHeader.name.lowercased().contains(searchText.lowercased()) {
                    return true
                }
            }
            return false
        }
    }
}
struct ListRowModifier: ViewModifier {
    func body(content: Content) -> some View {
        Group {
            content
            Divider()
        }
    }
}
```
Lastly we need to update `SubMenuView` to filter only those names which contains the searchText.

```swift
// To filter only those subHeader names containing searchText
 if let filteredSubItems = searchText.isEmpty ? header.subHeader : header.subHeader.filter({ (subHeader) -> Bool in
                    subHeader.name.lowercased().contains(searchText.lowercased())
 })
```

Hope above tutorial was informative and useful. You can check the source code on github : [HERE](https://github.com/iameetshah/ExpandableScrollViewWithSearch)
