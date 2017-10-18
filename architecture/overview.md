# E-Commerce System Architecture
everything between product import and order export

prepared January 30, 2003 by Jason Abbott

## User Interface
Accurate product information and on-time deliveries are the customer’s basic expectation.  Hence, though important, they are less likely to set us apart from our competitors.  The differentiator is site design and functionality.  The current design, implemented in January of 2002, adheres to rules meant to ensure intuitive navigation and comprehension of page elements.  Based on customer and press feedback, these rules have been very successful.

Site usability will be correlated with the intelligibility of navigation elements.  A problem with the previous site design was an apparent lack of organization and consistency with these elements.  Shopping links were scattered around the page, intermixed with links for managing one’s own account.

### Home Page
The current design sought to improve usability by grouping functionality and giving permanent visibility to links that a customer might use repeatedly, while moving less-used links to other areas, to reduce clutter.  **Table 1.1** is a diagram of our current site’s home page.  The highlighted areas are those that remain permanently during a customer’s shopping session.  The following rules apply:

- To qualify as a permanent link, the link should be one designed for repeated, non-contiguous use during a single shopping session.
- Links not so-designed should be relegated to other areas.
- To maintain intelligibility, links should remain segmented by high-level functionality.  A link to items on special would not belong, for example, in the administrative or cart/list areas. 
- For the same reason, link types should remain consistent.  For example, a link with the visible appearance of a tab that pops a menu should not be mixed with a link that looks the same but does something other than pop a menu.

The two non-permanent areas in the above diagram contain links to static pages.  What is meant by this are links to pages that the customer cannot modify and that will not change during a customer’s shopping session.  The pages may be dynamic in the sense that they are database driven, but they are still static from the shopper’s perspective.

Since the content of such pages is static, customer’s have no reason to visit them more than once during a shopping session, thus they do not qualify for placement in a permanent section.

### Remainder redacted