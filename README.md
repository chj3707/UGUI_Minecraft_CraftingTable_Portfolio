# Portfolio

#### [개발 인원] 1인
#### [구현 내용]
#### 마인크래프트 제작대 구현
+ [데이터 베이스](#데이터-베이스)
+ [인벤토리 아이템 추가](#인벤토리-아이템-추가)
+ [슬롯 내의 아이템 정보](#슬롯-내의-아이템-정보)
+ [슬롯 클릭 처리](#슬롯-클릭-처리)
+ [제작대 아이템 조합](#제작대-아이템-조합)

## 데이터 베이스
+ ### 아이템 :: ScriptableObject

    + *__ScriptableObject__ 생성을 위한 클래스*   

```c#
[CreateAssetMenu(fileName = "New Item", menuName = "New Item/Item")]
public class Item : ScriptableObject
{
    public string item_name;                      // 아이템 이름
    public EItemType item_type;                   // 아이템 타입
    public Sprite item_sprite;                    // 아이템 스프라이트
    public bool is_stackable;                     // 아이템을 쌓을 수 있는가?
    public bool is_material;                      // 재료 아이템인가?
}
```

<img src="/Image/ScriptableObject.PNG" height="60%" width="100%">

+ ### 아이템 레시피 :: CSV

    + _CSVReader 오픈 소스 사용_

```c#
public class ItemRecipe
{
    private string[,] recipe;       // 아이템 레시피
    private int create_quantity;    // 아이템 생성 개수
    private int material_quantity;  // 필요한 재료 개수

    public ItemRecipe()
    {
        recipe = new string[3, 3];
        create_quantity = 0;
        material_quantity = 0;
    }
    
    // ...프로퍼티
}
```

_**Recipe.csv**_

<img src="/Image/RecipeCSV.png" height="60%" width="100%">

+ ### ItemDataBase
    + _아이템, 아이템 레시피 정보 [SET, GET]_

```c#
public class ItemDataBase : Singleton_Mono<ItemDataBase>
{
    public Dictionary<string, Item> item_database;                      
    public List<Dictionary<string, ItemRecipe>> item_recipe_database;   
    
    // ... Awake :: Initialize, Set info
    // ... Initialize :: database 동적 할당

    public Item get_item_data(string item_name)
    {
        return item_database[item_name];
    }
    
    public Dictionary<string, ItemRecipe> get_item_recipe_data(int material_quantity)
    {
        return item_recipe_database[material_quantity];
    }

    // 아이템 정보 저장 :: ScriptableObject
    private void set_items_info()
    {
        Item[] items_info = Resources.LoadAll<Item>("ScriptableObject/Items");      
        int item_count = items_info.Length;                                        

        for (int i = 0; i < item_count; i++)
            item_database.Add(items_info[i].item_name, items_info[i]);           
    }

    // 아이템 레시피 정보 저장 :: CSV
    private void set_items_recipe_info()
    {
        List<Dictionary<string, object>> csv_data = CSVReader.Read("CSV/Recipe");
        int recipe_count = csv_data.Count;

        for (int i = 0; i < recipe_count; i++)
        {
            ItemRecipe curr_item_recipe = new ItemRecipe();  
            string item_name = "";                           

            item_name = csv_data[i]["조합 아이템"] as string;
            curr_item_recipe.CreateQuantity = int.Parse(csv_data[i]["생성 개수"].ToString());

            for (int j = 0; j < 3; j++)
            {
                for (int k = 0; k < 3; k++)
                {
                    string material_item_name = csv_data[i][$"\"{j},{k}\""] as string;
                    curr_item_recipe.Recipe[j, k] = string.IsNullOrEmpty(material_item_name) ? string.Empty : material_item_name;
                    curr_item_recipe.MaterialQuantity += string.IsNullOrEmpty(material_item_name) ? 0 : 1;                  
                }
            }

            item_recipe_database[curr_item_recipe.MaterialQuantity].Add(item_name, curr_item_recipe);
        }
    }
}

```

## 인벤토리 아이템 추가

<p align="center">
    <img src="/Image/Material Item Button.png" align="center" width="49%">
    <img src="/Image/Add Item to Inventory.png" align="center" width="49%">
    <figcaption align="center"></figcaption>
</p>

+ #### **_아이템 버튼 클릭_**

```c#
    public void _On_AddItemBtnClick()
    {
        GameObject current_click_btn = EventSystem.current.currentSelectedGameObject;     
        Item current_click_item = current_click_btn.GetComponent<Item_Scriptable>().item; 

        // 인벤토리에 클릭한 아이템 추가
        Inventory.insert_item_to_inventory(current_click_item);
    }
```

+ #### **_인벤토리 아이템 추가 로직_**
    + _1. 빈 슬롯_
    + _2. 추가할 아이템과 같은 아이템, 슬롯의 아이템 Not Full_
    + _3. 추가할 아이템의 남은 개수 1개 이상_
    + _-> 슬롯에 아이템 추가하고 다음 슬롯 확인_
```c#

    public static void insert_item_to_inventory(Item insert_item)
    {
        IEnumerator<Slot> enumerator = slot_list.GetEnumerator();
        int insert_item_count = insert_item.is_stackable ? MaxItemStack.stackable : MaxItemStack.non_stackable;

        while (enumerator.MoveNext())
        {
            Slot current_slot = enumerator.Current;                         
            Item current_item = current_slot.item_info.get_top_item_info(); 

            if (null == current_item)
            {
                for (int i = 0; i < insert_item_count; i++)
                    current_slot.item_info.item_stack.Push(insert_item);

                insert_item_count = 0;
            }
            else if(insert_item == current_item)
            {
                while (false == current_slot.item_info.is_item_stack_full() && insert_item_count > 0)
                {
                    current_slot.item_info.item_stack.Push(insert_item);
                    --insert_item_count;
                }
            }
            current_slot.item_info.update_UI();       
            if (0 == insert_item_count) break;        
        }
    }
```

## 슬롯 내의 아이템 정보

+ #### **_아이템 정보 클래스 사용처_**
    + _1. 인벤토리, 작업대 슬롯_
    + _2. 완성 아이템 슬롯_
    + _3. 드래그 아이템_
    
```c#
public class ItemInfo
{
    public Stack<Item> item_stack;
    public Image item_image;                  
    public Text item_quantity_text;

    public ItemInfo() { // ... 변수 초기화 }

    // 아이템 변경 내용 있을 때마다 호출하여 UI 업데이트
    public void update_UI()
    {
        if (true == is_item_stack_empty())
        {
            item_image.sprite = null;
            item_image.enabled = false;

            item_quantity_text.text = item_stack.Count.ToString();
            item_quantity_text.enabled = false;
            return;
        }

        Item current_item = get_top_item_info();
        item_image.enabled = true;
        item_image.sprite = current_item.item_sprite;
        item_quantity_text.enabled = current_item.is_stackable ? true : false;
        item_quantity_text.enabled = 1 == get_item_stack_quantity() ? false : true;
        item_quantity_text.text = item_stack.Count.ToString();
    }

    public int get_current_item_quantity() { // ... }
    
    public int get_item_max_quantity() { // ... }
    
    public Item get_item_info() { // ... }
    
    public string get_item_name() { // ... }
    
    public bool is_item_stack_full() { // ... }

    public bool is_item_stack_empty() { // ... }
}
```

## 슬롯 클릭 처리

+ #### **_인벤토리, 작업대 슬롯 클릭 IPointerClickHandler 사용_**
    + _슬롯 클릭 상황 별 처리_
        + _Non Dragging_
            + _빈 슬롯 :: 함수 종료_
            + [_아이템 슬롯 :: 아이템 드래그_](#아이템-드래그)
        + _Dragging_
            + [_빈 슬롯 :: 아이템 드랍_](#아이템-드랍)
            + [_아이템 슬롯 :: 아이템 드랍 or 스왑_](#아이템-드랍-or-스왑)

```c#
    public void OnPointerClick(PointerEventData eventdata)
    {
        EventManager eventmanager = EventManager.GetInstance;

        switch (item_info.is_item_stack_empty())
        {
            case true:                            // 빈 슬롯 클릭
                switch (eventmanager.is_dragging)
                {
                    case false:                   // NonDragging :: 함수 종료
                        return;
                    
                    case true:                    // Dragging :: 아이템 드랍
                        items_drop(eventdata, eventmanager);
                        break;
                }
                break;
                
            case false:                           // 아이템 슬롯 클릭
                switch (eventmanager.is_dragging)
                {
                    case true:                    // Dragging :: 아이템 드랍 or 스왑
                        items_swap_drop(eventdata, eventmanager);
                        break;
                        
                    case false:                   // NonDragging :: 아이템 드래그
                        items_drag(eventdata, eventmanager);    
                        break;
                }
                break;
        }

        // 작업대 슬롯 클릭 :: 조합 실행
        if (true == is_workbench_slot)
        {
            Workbench temp_workbench = this.GetComponentInParent<Workbench>();
            temp_workbench.compare_workbench_and_recipes();
        }
    }
```

### **_아이템 드래그_**
<img src="/Image/ItemDragging.gif" height="50%" width="50%">

+ **_좌 클릭 :: 아이템 전부 드래그_**
+ **_우 클릭 :: 아이템 절반 드래그_**

```c#
    private void items_drag(PointerEventData eventdata, EventManager eventmanager)
    {
        DraggingItem dragging_item = eventmanager.dragging_item_obj.GetComponent<DraggingItem>();
        int pickup_item_count = 0;

        switch (eventdata.pointerId)
        {
            case -1:                // 좌 클릭
                pickup_item_count = item_info.get_item_stack_quantity();
                break;

            case -2:                // 우 클릭
                pickup_item_count = Mathf.CeilToInt(item_info.get_item_stack_quantity() * 0.5f);
                break;
        }
        eventmanager.is_dragging = true;

        // 슬롯 아이템 Pop(), 드래그 아이템 Push()
        for (int i = 0; i < pickup_item_count; i++)
            dragging_item.item_info.item_stack.Push(this.item_info.item_stack.Pop());

        dragging_item.item_info.update_UI();
        this.item_info.update_UI();
    }
```

### **_아이템 드랍_**
<p align="center">
    <img src="/Image/ItemDrop.gif" align="center" width="49%">
    <img src="/Image/ItemDrop2.gif" align="center" width="49%">
    <figcaption align="center"></figcaption>
</p>

+ **_좌 클릭_**
    + _빈 슬롯 :: 아이템 전부 드랍_
    + _아이템 슬롯 :: 슬롯 아이템 최대 개수만큼 드랍_
    
+ **_우 클릭 :: 아이템 1개 드랍_**

```c#
    private void items_drop(PointerEventData eventdata, EventManager eventmanager)
    {
        DraggingItem dragging_item = eventmanager.dragging_item_obj.GetComponent<DraggingItem>();
        int drop_item_count = 0;

        switch (eventdata.pointerId)
        {
            case -1:                                              // 좌 클릭
                if (true == this.item_info.is_item_stack_empty()) // 빈 슬롯
                    drop_item_count = dragging_item.item_info.get_item_stack_quantity();
                    
                else                                              // 아이템 슬롯
                {
                    int dragging_item_count = dragging_item.item_info.get_item_stack_quantity();
                    int required_item_count = this.item_info.get_max_item_stack() - this.item_info.get_item_stack_quantity();

                    drop_item_count = dragging_item_count >= required_item_count ? required_item_count : dragging_item_count;
                }
                break;

            case -2:                                              // 우 클릭
                if (true == this.item_info.is_item_stack_empty() ||
                    this.item_info.get_item_stack_quantity() < this.item_info.get_max_item_stack())
                {
                    drop_item_count = 1;
                }
                break;
        }

        // 드래그 아이템 Pop(), 슬롯 아이템 Push()
        for (int i = 0; i < drop_item_count; i++)
            this.item_info.item_stack.Push(dragging_item.item_info.item_stack.Pop());

        dragging_item.item_info.update_UI();
        this.item_info.update_UI();

        if (true == dragging_item.item_info.is_item_stack_empty())
            eventmanager.is_dragging = false;
    }
```

### **_아이템 드랍 or 스왑_**
<img src="/Image/ItemSwap.gif" height="50%" width="50%">

+ **_같은 아이템 :: 아이템 드랍_**
+ **_다른 아이템 :: 아이템 스왑_**

```c#
    private void items_swap_drop(PointerEventData eventdata, EventManager eventmanager)
    {
        DraggingItem dragging_item = eventmanager.dragging_item_obj.GetComponent<DraggingItem>();

        // 같은 아이템 :: 아이템 드랍
        if (dragging_item.item_info.get_top_item_info() == this.item_info.get_top_item_info())
        {
            items_drop(eventdata, eventmanager); 
            return;
        }
        else // 다른 아이템 :: 아이템 데이터 스왑
        {
            Core.swap<Stack<Item>>(ref dragging_item.item_info.item_stack, ref this.item_info.item_stack);
            dragging_item.item_info.update_UI();
            this.item_info.update_UI();
        }
    }
```

## 제작대 아이템 조합
<p align="center">
    <img src="/Image/Crafting1.gif" align="center" width="49%">
    <img src="/Image/Crafting2.gif" align="center" width="49%">
    <figcaption align="center"></figcaption>
</p>

+ ### **_아이템 제작 플로우_**
    + _1. 제작대 슬롯 아이템 추가 및 제거 or 제작된 완성 아이템 클릭 -> [제작대와 레시피 탐색 시작](#제작대와-레시피-탐색)_
    + _2. 제작대에 올라간 아이템 개수로 ItemDataBase에서 아이템 레시피 데이터 가져오기_
    + _3. 제작대와 레시피 비교 -> [제작 가능한 아이템 완성품 슬롯에 생성](#완성-아이템-제작)_
    + _4. 완성 아이템 클릭 -> [제작대 슬롯의 재료 아이템 소모](#재료-아이템-소모)_

##### **_제작대와 레시피 탐색_**
```c#
    public void compare_workbench_with_recipes()
    {
        crafting_item_slot.reset_item_slot();
        var item_recipe_datas = ItemDataBase.GetInstance.get_item_recipe_data(workbench_material_quantity);

        foreach (var item_recipe in item_recipe_datas)
        {
            string crafting_item_name = item_recipe.Key;
            string[,] crafting_item_recipe = item_recipe.Value.Recipe;

            bool is_craftable = true;

            for (int i = 0; i < row; i++)
            {
                for (int j = 0; j < col; j++)
                {
                    string material_item_name = workbench[i, j].item_info.get_top_item_name();

                    if (true == is_common_material(material_item_name, ECommonMaterial.none))
                    {
                        string[] split_item_name = material_item_name.Split(' ');
                        string prefix = split_item_name[0];
                        string suffix = split_item_name[1];

                        if (crafting_item_recipe[i, j] != suffix) { is_craftable = false; break; }
                        if (true == is_common_material(crafting_item_name, ECommonMaterial.none)) { crafting_item_name = prefix + " " + crafting_item_name; }
                    }
                    else if (crafting_item_recipe[i, j] != material_item_name) { is_craftable = false; break; }
                }

                if (false == is_craftable) break;
            }

            if (true == is_craftable) { item_crafting(crafting_item_name, item_recipe.Value.CreateQuantity); break; }
        }
    }
```

##### **_완성 아이템 제작_**

```c#
    private void item_crafting(string crafting_item_name, int create_quantity)
    {
        Item crafting_item = ItemDataBase.GetInstance.get_item_data(crafting_item_name);

        for (int i = 0; i < create_quantity; i++)
            crafting_item_slot.item_info.item_stack.Push(crafting_item);

        crafting_item_slot.item_info.update_UI();
    }
```

##### **_재료 아이템 소모_**

```c#
    public void consume_material_item()
    {
        for (int i = 0; i < row; i++)
        {
            for (int j = 0; j < col; j++)
            {
                ItemInfo current_item_info = workbench[i, j].item_info;
                if (null == current_item_info.get_item_info()) continue;

                string material_item_name = search_nonconsume_material(current_item_info.get_item_name(), ENonConsumeMaterialItem.none);
                if (String.Empty == material_item_name)
                {
                    current_item_info.item_stack.Pop();
                    if (true == current_item_info.is_item_stack_empty()) --workbench_material_quantity;
                }
                else
                {
                    current_item_info.item_stack.Clear();
                    current_item_info.item_stack.Push(ItemDataBase.GetInstance.get_item_data(material_item_name));
                }
                current_item_info.update_UI();
            }
        }
    }
```
