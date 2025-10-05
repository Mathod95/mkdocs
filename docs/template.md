
## Introduction

### Objectifs

### Prérequis

### Ma configuration

---

## Title1

### Title2

#### Title3

##### Title4

###### Title5

---

## Conclusion

### En rapport avec cet article

### Liens utile

``` yaml
theme:
  features:
    - content.code.annotate # (1)
```

1.  :man_raising_hand: I'm a code annotation! I can contain `code`, __formatted
    text__, images, ... basically anything that can be written in Markdown.


=== "Python"

    !!! note "Note for Python"
        This is a note inside the Python tab.

=== "JavaScript"

    !!! warning "Warning for JavaScript"
        This is a warning inside the JavaScript tab.

!!! note "Parent Admonition"

    This is the outer admonition content.

    === "Tab 1"

        !!! tip "Tip inside Tab 1"
            This is the first tab's inner admonition.

    === "Tab 2"

        !!! warning "Warning inside Tab 2"
            This is the second tab's inner admonition.



https://www.youtube.com/watch?v=pPEUhfTZswc