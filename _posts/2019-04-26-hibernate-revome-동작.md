---
title: "[Spring/Project] Repository Delete시 동작과 child entity 삭제"
date: 2019-04-26 02:25:28 -0400
categories: Java/Spring
---

# 개요

카테고리 구조의 데이터에서 카테고리 세부내용 하나를 삭제할 경우. 정상적으로 삭제되지 않는 상황이 발생한다.



**Category Entity**

```java

@Entity
@Getter
@Setter
@NoArgsConstructor
public class MenuCategory extends AbstractEntity {

    @Size(min = 1, max = 50)
    private String name;

    @ManyToOne
    @JoinColumn(name = "parentId")
    @JsonIgnore
    private MenuCategory parent;

    @OneToMany(cascade = CascadeType., fetch = FetchType.EAGER)
    @JoinColumn(name = "parentId")
    private List<MenuCategory> children = new ArrayList<>();

    public MenuCategory(Long id, String name) {
        super(id);
        this.name = name;
    }

    public MenuCategory(Long id, @Size(min = 1, max = 50) String name, MenuCategory parent, List<MenuCategory> children) {
        super(id);
        this.name = name;
        this.parent = parent;
        this.children = children;
    }

    public MenuCategory addChild(MenuCategory child) {
        children.add(child);
        child.setParent(this);
        return this;
    }

    public void removeChild(MenuCategory menuCategory) {
        children.remove(menuCategory);
    }

    @Override
    public String toString() {
        return "MenuCategory{" +
                "id=" + getId() +
                ", name='" + name + '\'' +
                ", children=" + children +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if(o instanceof MenuCategory){
            MenuCategory targetMenuCategory = (MenuCategory) o;
            return this.getId().equals(targetMenuCategory.getId());
        }
        return false;
    }
}
```



**MenuCategoryService.deleteWithChildren()**

```Java
 private MenuCategory deleteWithChildren(MenuCategory menuCategory) {

        MenuCategory parentCategory = menuCategory.getParent();
        parentCategory.removeChild(menuCategory);
        categoryRepository.save(parentCategory);
        categoryRepository.delete(menuCategory);
        return menuCategory;
    }
```



**실행결과**

![image-20190427015745754](/assets/images/image-20190427015745754.png)

현재 카테고리 세부사항중 하나를 삭제할 경우.



> 공부하고 블로깅 :<https://whiteship.tistory.com/1457>