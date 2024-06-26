---
parser: v2
auto_validation: true
primary_tag: software-product-function>sap-s-4hana-cloud--abap-environment
tags:  [ tutorial>beginner, software-product-function>sap-s-4hana-cloud--abap-environment, programming-tool>abap-development, programming-tool>abap-extensibility ] 
time: 25
author_name: Merve Temel
author_profile: https://github.com/mervey45
--- 
 
# Integrate released purchase requisition API into Shopping Cart Business Object
<!-- description --> Integrate released purchase requisition API into shopping cart business object with SAP S/4HANA Cloud, ABAP Environment or SAP S/4HANA on-premise.
 
In the shopping cart, customers can order various items. Once an item is ordered, a new purchase requisition is created via purchase requisitions API.

[EML](https://help.sap.com/viewer/923180ddb98240829d935862025004d6/Cloud/en-US/af7782de6b9140e29a24eae607bf4138.html) can be used to trigger purchase requisitions API, which is released for cloud development.


## Prerequisites  
- This tutorial can be used in both SAP S/4HANA Cloud, private edition system and SAP S/4HANA on-premise system with release 2022 FPS01. We suggest using a [Fully-Activated Appliance](https://blogs.sap.com/2018/12/12/sap-s4hana-fully-activated-appliance-create-your-sap-s4hana-1809-system-in-a-fraction-of-the-usual-setup-time/) in SAP Cloud Appliance Library for an easy start without the need for system setup.
- For SAP S/4HANA on-premise, create developer user with full development authorization 
- You have installed the latest [Eclipse with ADT](abap-install-adt).
- Business role `SAP_BR_PURCHASER` needs to be assigned to your business user
- Use Starter Development Tenant in S/4HANA Cloud for the tutorial to have necessary sample data in place. See [3-System Landscape and Transport Management](https://help.sap.com/docs/SAP_S4HANA_CLOUD/a630d57fc5004c6383e7a81efee7a8bb/e022623ec1fc4d61abb398e411670200.html?state=DRAFT&version=2208.503).


## You will learn  
- How to enhance the behavior definition of a data model 
- How to enhance the behavior definition of a projection view
- How to enhance metadata extensions
- How to implement the unmanaged save and `save_modified`
- How to enhance a behavior implementation 
- How to run the SAP Fiori Elements Preview 
- How to check purchase requisitions 

## Intro
In this tutorial, wherever `###` appears, use a number (e.g. 000).

---


### Enhance behavior definition of data model 

**Hint:** In case of S/4HANA 2022 `FPS01`, strict(1) mode must be used.

  1. Add the following action statement for `createPurchaseRequisitionItem ` to your behavior definition **`ZR_SHOPCARTTP_###`**.

    ```ABAP
    action(features:instance) createPurchaseRequisitionItem  result [1] $self;
    ```
   
     ![projection](bdef5.png)

     **Hint:** Please replace **`###`** with your ID. 

  2. Check your behavior definition:

    ```ABAP
    managed implementation in class ZBP_SHOPCARTTP_### unique;
    strict ( 2 );
    with draft;

    define behavior for ZR_SHOPCARTTP_### alias ShoppingCart
    persistent table zashopcart_###
    draft table ZDSHOPCART_###
    etag master LocalLastChangedAt
    lock master total etag LastChangedAt
    authorization master( global )
    {
      field ( readonly )
      OrderUUID,
      CreatedAt,
      CreatedBy,
      LastChangedAt,
      LastChangedBy,
      LocalLastChangedAt,
      PurchaseRequisition,
      PrCreationDate,
      DeliveryDate;

      field ( numbering : managed )
      OrderUUID;


      create;
      update(features: instance) ;
      delete;

      draft action(features: instance) Edit;
      draft action Activate;
      draft action Discard;
      draft action Resume;
      draft determine action Prepare { validation checkOrderedQuantity;  validation checkDeliveryDate;}
        determination setInitialOrderValues on modify { create; }
        determination calculateTotalPrice on modify { create; field Price; } 
      validation checkOrderedQuantity on save { create; field OrderQuantity; }
      validation checkDeliveryDate on save { create; field DeliveryDate; }

    action(features:instance) createPurchaseRequisitionItem  result [1] $self;
      mapping for ZASHOPCART_### 
      {
        OrderUUID = order_uuid;
        OrderID = order_id;
        OrderedItem = ordered_item;
        Price = price;
        TotalPrice = total_price;
        Currency = currency;
        OrderQuantity = order_quantity;
        DeliveryDate = delivery_date;
        OverallStatus = overall_status;
        Notes = notes;
        CreatedBy = created_by;
        CreatedAt = created_at;
        LastChangedBy = last_changed_by;
        LastChangedAt = last_changed_at;
        LocalLastChangedAt = local_last_changed_at;
        PurchaseRequisition = purchase_requisition;
        PrCreationDate = pr_creation_date;
      }
    }
    ```

   3. Save and activate. 

### Enhance behavior definition of projection view


  1. Open your behavior definition **`ZC_SHOPCARTTP_###`** to enhance it. Add action `createPurchaseRequisitionItem` to your behavior definition.

    ```ABAP
    use action createPurchaseRequisitionItem;
    ```
    
    ![projection](actionnew.png)

  2. Check your behavior definition:

    ```ABAP
    projection;
    strict ( 2 );
    use draft;

    define behavior for ZC_SHOPCARTTP_### alias ShoppingCart
    use etag
    {
      use create;
      use update;
      use delete;

      use action Edit;
      use action Activate;
      use action Discard;
      use action Resume;
      use action Prepare;
      use action createPurchaseRequisitionItem;
    }
    ```

   3. Save and activate.


### Enhance metadata extension

  1. Open your metadata extension **`ZC_SHOPCARTTP_###`** to enhance it. Add following annotation to `PurchaseRequisition`.

    ```ABAP
    ,

    { type: #FOR_ACTION, dataAction: 'createPurchaseRequisitionItem', label: 'Create purchase requisition item' } ] 
    @UI.identification: [ {
      position: 100 , 
      label: 'PurchaseRequisition'
    }]
    PurchaseRequisition;
    ```

    ![projection](metadata.png)

  2. Check your metadata extension:

    ```ABAP
    @Metadata.layer: #CUSTOMER
    @UI: {
      headerInfo: {
        typeName: 'ShoppingCart', 
        typeNamePlural: 'ShoppingCart'
      }
    }

    annotate view ZC_SHOPCARTTP_### with
    {
      @UI.facet: [ {
        id: 'idIdentification', 
        type: #IDENTIFICATION_REFERENCE, 
        label: 'ShoppingCart', 
        position: 10 
      } ]
      @UI.hidden: true
      OrderUUID;
      
      @UI.lineItem: [ {
        position: 10 , 
        importance: #MEDIUM, 
        label: 'OrderID'
      } ]
      @UI.identification: [ {
        position: 10 , 
        label: 'OrderID'
      } ]
      OrderID;
      
      @UI.lineItem: [ {
        position: 20 , 
        importance: #MEDIUM, 
        label: 'OrderedItem'
      } ]
      @UI.identification: [ {
        position: 20 , 
        label: 'OrderedItem'
      } ]
      OrderedItem;
      
      @UI.lineItem: [ {
        position: 30 , 
        importance: #MEDIUM, 
        label: 'Price'
      } ]
      @UI.identification: [ {
        position: 30 , 
        label: 'Price'
      } ]
      Price;
      
      @UI.lineItem: [ {
        position: 40 , 
        importance: #MEDIUM, 
        label: 'TotalPrice'
      } ]
      @UI.identification: [ {
        position: 40 , 
        label: 'TotalPrice'
      } ]
      TotalPrice;
      
      @UI.lineItem: [ {
        position: 50 , 
        importance: #MEDIUM, 
        label: 'Currency'
      } ]
      @UI.identification: [ {
        position: 50 , 
        label: 'Currency'
      } ]
      Currency;
      
      @UI.lineItem: [ {
        position: 60 , 
        importance: #MEDIUM, 
        label: 'OrderQuantity'
      } ]
      @UI.identification: [ {
        position: 60 , 
        label: 'OrderQuantity'
      } ]
      OrderQuantity;
      
      @UI.lineItem: [ {
        position: 70 , 
        importance: #MEDIUM, 
        label: 'DeliveryDate'
      } ]
      @UI.identification: [ {
        position: 70 , 
        label: 'DeliveryDate'
      } ]
      DeliveryDate;
      
      @UI.lineItem: [ {
        position: 80 , 
        importance: #MEDIUM, 
        label: 'OverallStatus'
      } ]
      @UI.identification: [ {
        position: 80 , 
        label: 'OverallStatus'
      } ]
      OverallStatus;
      
      @UI.lineItem: [ {
        position: 90 , 
        importance: #MEDIUM, 
        label: 'Notes'
      } ]
      @UI.identification: [ {
        position: 90 , 
        label: 'Notes'
      } ]
      Notes;
      
      @UI.hidden: true
      LocalLastChangedAt;
      
      @UI.lineItem: [ {
        position: 100 , 
        importance: #MEDIUM, 
        label: 'PurchaseRequisition'
      },

      { type: #FOR_ACTION, dataAction: 'createPurchaseRequisitionItem', label: 'Create purchase requisition item' } ] 
      @UI.identification: [ {
        position: 100 , 
        label: 'PurchaseRequisition'
      }]
      PurchaseRequisition;
      
      @UI.lineItem: [ {
        position: 110 , 
        importance: #MEDIUM, 
        label: 'PrCreationDate'
      } ]
      @UI.identification: [ {
        position: 110 , 
        label: 'PrCreationDate'
      } ]
      PrCreationDate;
    }
    ```

   3. Save and activate.
 
### Implement unmanaged save and save_modified

In our scenario, we want to integrate the released purchase requisition API during the save sequence.

  1. Open the behavior definition `ZR_SHOPCARTTP_###`, delete the following line:
  
    ```ABAP
    persistent table zashopcart_### 
    ```
  
  2. In your behavior definition **`ZR_SHOPCARTTP_###`** add the `with unmanaged save` statement.

    ```ABAP
    with unmanaged save
    ```
    So that it looks as follows:

    ```ABAP
    managed implementation in class ZBP_SHOPCARTTP_### unique;
    strict ( 2 );
    with draft;

    define behavior for ZR_SHOPCARTTP_### alias ShoppingCart
    draft table ZDSHOPCART_###
    etag master LocalLastChangedAt
    lock master total etag LastChangedAt
    authorization master( global )
    with unmanaged save
    {
      field ( readonly )
      OrderUUID,
      CreatedAt,
      CreatedBy,
      LastChangedAt,
      LastChangedBy,
      LocalLastChangedAt,
      PurchaseRequisition,
      PrCreationDate,
      DeliveryDate;

      field ( numbering : managed )
      OrderUUID;

      create;
      update(features: instance) ;
      delete;

      draft action(features: instance) Edit;
      draft action Activate;
      draft action Discard;
      draft action Resume;
      draft determine action Prepare { validation checkOrderedQuantity;  validation checkDeliveryDate;}
        determination setInitialOrderValues on modify { create; }
        determination calculateTotalPrice on modify { create; field Price; } 
      validation checkOrderedQuantity on save { create; field OrderQuantity; }
      validation checkDeliveryDate on save { create; field DeliveryDate; }

    action(features:instance) createPurchaseRequisitionItem  result [1] $self;
      mapping for ZASHOPCART_### 
      {
        OrderUUID = order_uuid;
        OrderID = order_id;
        OrderedItem = ordered_item;
        Price = price;
        TotalPrice = total_price;
        Currency = currency;
        OrderQuantity = order_quantity;
        DeliveryDate = delivery_date;
        OverallStatus = overall_status;
        Notes = notes;
        CreatedBy = created_by;
        CreatedAt = created_at;
        LastChangedBy = last_changed_by;
        LastChangedAt = last_changed_at;
        LocalLastChangedAt = local_last_changed_at;
        PurchaseRequisition = purchase_requisition;
        PrCreationDate = pr_creation_date;
      }
    }
    ```

  4. Save and activate it. Position the cursor on the with unmanaged save statement and use the shortcut ctrl + 1 to load the quick assist proposals, then double-click on Add required method `save_modified` in new local saver class to automatically create an empty implementation for the method. Implement it as follows:

    ```ABAP
      METHOD save_modified.

        DATA : lt_shopping_cart_as        TYPE STANDARD TABLE OF zashopcart_###,
              ls_shopping_cart_as        TYPE                   zashopcart_###.
        IF create-shoppingcart IS NOT INITIAL.
          lt_shopping_cart_as = CORRESPONDING #( create-shoppingcart MAPPING FROM ENTITY ).
          INSERT zashopcart_### FROM TABLE @lt_shopping_cart_as.
        ENDIF.
        IF update IS NOT INITIAL.
          CLEAR lt_shopping_cart_as.
          lt_shopping_cart_as = CORRESPONDING #( update-shoppingcart MAPPING FROM ENTITY ).
          LOOP AT update-shoppingcart  INTO DATA(shoppingcart) WHERE OrderUUID IS NOT INITIAL.
    *           select * from zashopcart_###where order_uuid = @shoppingcart-OrderUUID into @data(ls_shoppingcart) .
    *                      lt_shopping_cart_as = CORRESPONDING #( create-shoppingcart MAPPING FROM ENTITY ).

            MODIFY zashopcart_### FROM TABLE @lt_shopping_cart_as.
    *           ENDSELECT.
          ENDLOOP.
        ENDIF.

        LOOP AT delete-shoppingcart INTO DATA(shoppingcart_delete) WHERE OrderUUID IS NOT INITIAL.
          DELETE FROM zashopcart_### WHERE order_uuid = @shoppingcart_delete-OrderUUID.
          DELETE FROM zdshopcart_### WHERE orderuuid = @shoppingcart_delete-OrderUUID.
        ENDLOOP.
      ENDMETHOD.

    ```

   5. Save and activate it.

>We use the unmanaged save option for our scenario, rather than the additional save option. This is because the additional save should only be used in case data needs to be saved in addition to BO data in a persistence outside the BO, as stated in the [Additional Save documentation](https://help.sap.com/docs/SAP_S4HANA_CLOUD/e5522a8a7b174979913c99268bc03f1a/ca7097c8ea404b11b1f1334fd54cdd15.html). Since this is not our use case (the purchase requisition is created and saved in the persistency of the shopping cart BO), we rely on the unmanaged save option.


### Enhance behavior implementation

**Hint:** Please replace **`###`** with your ID. 

**For S/4HANA on premise:** Material `D001` or similar needs to be created in system if S/4HANA on-premise is used. [More information](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/f7fddfe4caca43dd967ac4c9ce6a70e4/23d6b8535c39b44ce10000000a174cb4.html?version=2022.000)


  1. In your **Global Class**, replace your code with following:

    ```ABAP
    class ZBP_SHOPCARTTP_### DEFINITION PUBLIC ABSTRACT FINAL FOR BEHAVIOR OF zr_shopcarttp_###.
    PUBLIC SECTION.
      CLASS-DATA mapped_purchase_requisition TYPE response for mapped i_purchaserequisitiontp.

    ENDCLASS.


    CLASS zbp_shopcarttp_### IMPLEMENTATION. 
    ENDCLASS.
    ```

  2. Add the missing constant to your implementation, the `createPurchaseRequisitionItem` action and enhance the `save_modified` method. In your **Local Types**, replace your code with following:

    ```ABAP
    CLASS lhc_ShopCart DEFINITION INHERITING FROM cl_abap_behavior_handler.
      PRIVATE SECTION.
        CONSTANTS:
          BEGIN OF is_draft,
            false TYPE abp_behv_flag VALUE '00', " active (not draft)
            true  TYPE abp_behv_flag VALUE '01', " draft
          END OF is_draft.
        CONSTANTS:
          BEGIN OF c_overall_status,
            new       TYPE string VALUE 'New / Composing',
    *         composing  type string value 'Composing...',
            submitted TYPE string VALUE 'Submitted / Approved',
            cancelled TYPE string VALUE 'Cancelled',
          END OF c_overall_status.
        METHODS: get_global_authorizations FOR GLOBAL AUTHORIZATION
          IMPORTING REQUEST requested_authorizations FOR ShoppingCart RESULT result,
        createPurchaseRequisitionItem FOR MODIFY
          IMPORTING keys FOR ACTION shoppingcart~createPurchaseRequisitionItem RESULT result,
          get_instance_features FOR INSTANCE FEATURES
            IMPORTING keys REQUEST requested_features FOR ShoppingCart RESULT result,
                setInitialOrderValues FOR DETERMINE ON MODIFY
                      IMPORTING keys FOR shoppingcart~setInitialOrderValues,
                calculateTotalPrice FOR DETERMINE ON MODIFY
                      IMPORTING keys FOR shoppingcart~calculateTotalPrice.

        METHODS checkDeliveryDate FOR VALIDATE ON SAVE
          IMPORTING keys FOR shoppingcart~checkDeliveryDate.

        METHODS checkOrderedQuantity FOR VALIDATE ON SAVE
          IMPORTING keys FOR shoppingcart~checkOrderedQuantity.

    ENDCLASS.

    CLASS lhc_ShopCart IMPLEMENTATION.

      METHOD get_global_authorizations.
      ENDMETHOD.

      METHOD createPurchaseRequisitionItem.
        DATA: purchase_requisitions      TYPE TABLE FOR CREATE I_PurchaserequisitionTP,
              purchase_requisition       TYPE STRUCTURE FOR CREATE I_PurchaserequisitionTP,
              purchase_requisition_items TYPE TABLE FOR CREATE i_purchaserequisitionTP\_PurchaseRequisitionItem,
              purchase_requisition_item  TYPE STRUCTURE FOR CREATE i_purchaserequisitiontp\\purchaserequisition\_purchaserequisitionitem,
              purchase_reqn_acct_assgmts TYPE TABLE FOR CREATE I_PurchaseReqnItemTP\_PurchaseReqnAcctAssgmt,
              purchase_reqn_acct_assgmt  TYPE STRUCTURE FOR CREATE I_PurchaseReqnItemTP\_PurchaseReqnAcctAssgmt,
              purchase_reqn_item_texts   TYPE TABLE FOR CREATE I_PurchaseReqnItemTP\_PurchaseReqnItemText,
              purchase_reqn_item_text    TYPE STRUCTURE FOR CREATE I_PurchaseReqnItemTP\_PurchaseReqnItemText,
              update_lines               TYPE TABLE FOR UPDATE zr_shopcarttp_###\\ShoppingCart,
              update_line                TYPE STRUCTURE FOR UPDATE zr_shopcarttp_###\\ShoppingCart,
              delivery_date              TYPE I_PurchaseReqnItemTP-DeliveryDate,
              requested_quantity         TYPE I_PurchaseReqnItemTP-RequestedQuantity.

    *    delivery_date = cl_abap_context_info=>get_system_date(  ) + 14.

        "read transfered order instances
        READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            ALL FIELDS WITH
            CORRESPONDING #( keys )
          RESULT DATA(OnlineOrders).

        DATA n TYPE i.

        LOOP AT OnlineOrders INTO DATA(OnlineOrder).
          n += 1.
          "purchase requisition
          purchase_requisition = VALUE #(  %cid                   = |My%CID_{ n }|
                                            purchaserequisitiontype  = 'NB'  ) .
          APPEND purchase_requisition TO purchase_requisitions.

          "purchase requisition item
          purchase_requisition_item = VALUE #(
                                            %cid_ref = |My%CID_{ n }|
                                            %target  = VALUE #(  (
                                                          %cid                         = |My%ItemCID_{ n }|
                                                          plant                        = '1010'  "Plant 01 (DE)
                                                          accountassignmentcategory    = 'U'  "unknown
    *                                                       PurchaseRequisitionItemText  =  . "retrieved automatically from maintained MaterialInfo
          requestedquantity            = OnlineOrder-OrderQuantity
          purchaserequisitionprice     = OnlineOrder-Price
          purreqnitemcurrency          = OnlineOrder-Currency
          Material                     = 'D001'
          materialgroup                = 'A001'
          purchasinggroup              = '001'
          purchasingorganization       = '1010'
          DeliveryDate                 = OnlineOrder-DeliveryDate   "delivery_date  "yyyy-mm-dd (at least 10 days)
          CreatedByUser                = OnlineOrder-CreatedBy
          ) ) ).
          APPEND purchase_requisition_item TO purchase_requisition_items.

          "purchase requisition account assignment
          purchase_reqn_acct_assgmt = VALUE #(
                                              %cid_ref = |My%ItemCID_{ n }|
                                              %target  = VALUE #( (
                                                            %cid       = |My%AccntCID_{ n }|
                                                            CostCenter = 'JMW-COST'
                                                            GLAccount  = '0000400000' ) ) ) .
          APPEND purchase_reqn_acct_assgmt TO purchase_reqn_acct_assgmts .

          "purchase requisition item text
          purchase_reqn_item_text    =  VALUE #(
                                              %cid_ref = |My%ItemCID_{ n }|
                                              %target  = VALUE #( (
                                                            %cid           = |My%TextCID_{ n }|
                                                            textobjecttype = 'B01'
                                                            language       = 'E'
                                                            plainlongtext  = OnlineOrder-Notes
                                                        )  )  ) .
          APPEND purchase_reqn_item_text TO purchase_reqn_item_texts.
        ENDLOOP.

        "EML deep create statement
        IF keys IS NOT INITIAL.
          "purchase reqn
          MODIFY ENTITIES OF i_purchaserequisitiontp
            ENTITY purchaserequisition
              CREATE FIELDS ( purchaserequisitiontype )
              WITH purchase_requisitions
            "purchase reqn item
            CREATE BY \_purchaserequisitionitem
              FIELDS ( plant
    *                   purchaserequisitionitemtext
                      accountassignmentcategory
                      requestedquantity
                      baseunit
                      purchaserequisitionprice
                      purreqnitemcurrency
                      Material
                      materialgroup
                      purchasinggroup
                      purchasingorganization
                      DeliveryDate
                    )
            with purchase_requisition_items
          "purchase reqn account assignment
          entity purchaserequisitionitem
            create by \_purchasereqnacctassgmt
                FIELDS ( CostCenter
                        GLAccount
                        Quantity
                        BaseUnit )
                WITH purchase_reqn_acct_assgmts
            "purchase reqn item text
            CREATE BY \_purchasereqnitemtext
                FIELDS ( plainlongtext )
                WITH purchase_reqn_item_texts
          REPORTED DATA(reported_create_pr)
          MAPPED DATA(mapped_create_pr)
          FAILED DATA(failed_create_pr).
        ENDIF.
        "retrieve the generated
        zbp_shopcarttp_###=>mapped_purchase_requisition-purchaserequisition = mapped_create_pr-purchaserequisition.

        "set a flag to check in the save sequence that purchase requisition has been created
        "the correct value for PurchaseRequisition has to be calculated in the save sequence using convert key
        LOOP AT keys INTO DATA(key).
          update_line-%tky           = key-%tky.
          update_line-OverallStatus  = c_overall_status-submitted. "'Submitted / Approved'.
          APPEND update_line TO update_lines.
        ENDLOOP.

        MODIFY ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            UPDATE
            FIELDS (  OverallStatus  )
            WITH update_lines
          REPORTED reported
          FAILED failed
          MAPPED mapped.

        IF failed IS INITIAL.
          "Read the changed data for action result
          READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
            ENTITY ShoppingCart
              ALL FIELDS WITH
              CORRESPONDING #( keys )
            RESULT DATA(result_read).
          "return result entities
          result = VALUE #( FOR order_2 IN result_read ( %tky   = order_2-%tky
                                                        %param = order_2 ) ).
        ENDIF.
      ENDMETHOD.

      METHOD get_instance_features.
        " read relevant olineShop instance data
        READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            FIELDS ( OverallStatus )
            WITH CORRESPONDING #( keys )
          RESULT DATA(OnlineOrders)
          FAILED failed.

        "ToDo: dynamic feature control is currently not working for the action cancel order

        " evaluate condition, set operation state, and set result parameter
        " update and checkout shall not be allowed as soon as purchase requisition has been created
        result = VALUE #( FOR OnlineOrder IN OnlineOrders
                          ( %tky                   = OnlineOrder-%tky
                            %features-%action-createPurchaseRequisitionItem
                              = COND #( WHEN OnlineOrder-OverallStatus = c_overall_status-submitted THEN if_abap_behv=>fc-o-disabled
                                        WHEN OnlineOrder-%is_draft = is_draft-true THEN if_abap_behv=>fc-o-disabled
                                        WHEN OnlineOrder-OverallStatus = c_overall_status-cancelled THEN if_abap_behv=>fc-o-disabled
                                        ELSE if_abap_behv=>fc-o-enabled   )
                            %features-%update
                              = COND #( WHEN OnlineOrder-OverallStatus = c_overall_status-submitted  THEN if_abap_behv=>fc-o-disabled
                                        WHEN OnlineOrder-OverallStatus = c_overall_status-cancelled THEN if_abap_behv=>fc-o-disabled
                                        ELSE if_abap_behv=>fc-o-enabled   )
    *                         %features-%delete
    *                           = COND #( WHEN OnlineOrder-PurchaseRequisition IS NOT INITIAL THEN if_abap_behv=>fc-o-disabled
    *                                     when OnlineOrder-OverallStatus = c_overall_status-cancelled then if_abap_behv=>fc-o-disabled
    *                                     else if_abap_behv=>fc-o-enabled   )
                            %action-Edit
                              = cond #( when OnlineOrder-OverallStatus = c_overall_status-submitted then if_abap_behv=>fc-o-disabled
                                        when OnlineOrder-OverallStatus = c_overall_status-cancelled then if_abap_behv=>fc-o-disabled
                                        else if_abap_behv=>fc-o-enabled   )

                            ) ).

      ENDMETHOD.
      METHOD setInitialOrderValues.
        DATA delivery_date TYPE I_PurchaseReqnItemTP-DeliveryDate.
        DATA(creation_date) = cl_abap_context_info=>get_system_date(  ).
        "set delivery date proposal
        delivery_date = cl_abap_context_info=>get_system_date(  ) + 14.
        "read transfered instances
        READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            FIELDS ( OrderID OverallStatus  DeliveryDate )
            WITH CORRESPONDING #( keys )
          RESULT DATA(OnlineOrders).

        "delete entries with assigned order ID
        DELETE OnlineOrders WHERE OrderID IS NOT INITIAL.
        CHECK OnlineOrders IS NOT INITIAL.

        " **Dummy logic to determine order IDs**
        " get max order ID from the relevant active and draft table entries
        SELECT MAX( order_id ) FROM zashopcart_### INTO @DATA(max_order_id). "active table
        SELECT SINGLE FROM zdshopcart_### FIELDS MAX( orderid ) INTO @DATA(max_orderid_draft). "draft table
        IF max_orderid_draft > max_order_id.
          max_order_id = max_orderid_draft.
        ENDIF.

        "set initial values of new instances
        MODIFY ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            UPDATE FIELDS ( OrderID OverallStatus  DeliveryDate Price  )
            WITH VALUE #( FOR order IN OnlineOrders INDEX INTO i (
                              %tky          = order-%tky
                              OrderID       = max_order_id + i
                              OverallStatus = c_overall_status-new  "'New / Composing'
                              DeliveryDate  = delivery_date
                              CreatedAt     = creation_date
                            ) ).
      ENDMETHOD.

      METHOD checkOrderedQuantity.
        "read relevant order instance data
        READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
        ENTITY ShoppingCart
        FIELDS ( OrderID OrderedItem OrderQuantity )
        WITH CORRESPONDING #( keys )
        RESULT DATA(OnlineOrders).

        "raise msg if 0 > qty <= 10
        LOOP AT OnlineOrders INTO DATA(OnlineOrder).
          APPEND VALUE #(  %tky           = OnlineOrder-%tky
                          %state_area    = 'VALIDATE_QUANTITY'
                        ) TO reported-shoppingcart.

          IF OnlineOrder-OrderQuantity IS INITIAL OR OnlineOrder-OrderQuantity = ' '.
            APPEND VALUE #( %tky = OnlineOrder-%tky ) TO failed-shoppingcart.
            APPEND VALUE #( %tky          = OnlineOrder-%tky
                            %state_area   = 'VALIDATE_QUANTITY'
                            %msg          = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-error
                                    text     = 'Quantity cannot be empty' )
                            %element-orderquantity = if_abap_behv=>mk-on
                          ) TO reported-shoppingcart.

          ELSEIF OnlineOrder-OrderQuantity > 10.
            APPEND VALUE #(  %tky = OnlineOrder-%tky ) TO failed-shoppingcart.
            APPEND VALUE #(  %tky          = OnlineOrder-%tky
                            %state_area   = 'VALIDATE_QUANTITY'
                            %msg          = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-error
                                    text     = 'Quantity should be below 10' )

                            %element-orderquantity  = if_abap_behv=>mk-on
                          ) TO reported-shoppingcart.
          ENDIF.
        ENDLOOP.
      ENDMETHOD.

      METHOD calculateTotalPrice.
        DATA total_price TYPE zr_shopcarttp_###-TotalPrice.

        " read transfered instances
        READ ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            FIELDS ( OrderID TotalPrice )
            WITH CORRESPONDING #( keys )
          RESULT DATA(OnlineOrders).

        LOOP AT OnlineOrders ASSIGNING FIELD-SYMBOL(<OnlineOrder>).
          " calculate total value
          <OnlineOrder>-TotalPrice = <OnlineOrder>-Price * <OnlineOrder>-OrderQuantity.
        ENDLOOP.

        "update instances
        MODIFY ENTITIES OF zr_shopcarttp_### IN LOCAL MODE
          ENTITY ShoppingCart
            UPDATE FIELDS ( TotalPrice )
            WITH VALUE #( FOR OnlineOrder IN OnlineOrders (
                              %tky       = OnlineOrder-%tky
                              TotalPrice = <OnlineOrder>-TotalPrice
                            ) ).
      ENDMETHOD.

      METHOD checkdeliverydate.
    *   " read transfered instances
            read entities of zr_shopcarttp_### in local mode
              entity ShoppingCart
                fields ( DeliveryDate )
                with corresponding #( keys )
              result data(OnlineOrders).

        DATA(creation_date) = cl_abap_context_info=>get_system_date(  ).
        "raise msg if 0 > qty <= 10
        LOOP AT OnlineOrders INTO DATA(online_order).


          IF online_order-DeliveryDate IS INITIAL OR online_order-DeliveryDate = ' '.
            APPEND VALUE #( %tky = online_order-%tky ) TO failed-shoppingcart.
            APPEND VALUE #( %tky         = online_order-%tky
                            %state_area   = 'VALIDATE_DELIVERYDATE'
                            %msg          = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-error
                                    text     = 'Delivery Date cannot be initial' )
                          ) TO reported-shoppingcart.

          ELSEIF  ( ( online_order-DeliveryDate ) - creation_date ) < 14.
            APPEND VALUE #(  %tky = online_order-%tky ) TO failed-shoppingcart.
            APPEND VALUE #(  %tky          = online_order-%tky
                            %state_area   = 'VALIDATE_DELIVERYDATE'
                            %msg          = new_message_with_text(
                                    severity = if_abap_behv_message=>severity-error
                                    text     = 'Delivery Date should be atleast 14 days after the creation date'  )

                            %element-orderquantity  = if_abap_behv=>mk-on
                          ) TO reported-shoppingcart.
          ENDIF.
        ENDLOOP.
      ENDMETHOD.

    ENDCLASS.

    CLASS lsc_ZR_SHOPCARTTP_### DEFINITION INHERITING FROM cl_abap_behavior_saver.
      PROTECTED SECTION.

        METHODS save_modified REDEFINITION.

        METHODS cleanup_finalize REDEFINITION.

    ENDCLASS.

    CLASS lsc_ZR_SHOPCARTTP_### IMPLEMENTATION.

      METHOD save_modified.
        DATA : lt_shopping_cart_as TYPE STANDARD TABLE OF zashopcart_###,
              ls_shopping_cart_as TYPE                   zashopcart_###.
        IF create-shoppingcart IS NOT INITIAL.
          lt_shopping_cart_as = CORRESPONDING #( create-shoppingcart MAPPING FROM ENTITY ).
          INSERT zashopcart_### FROM TABLE @lt_shopping_cart_as.
        ENDIF.
        IF update IS NOT INITIAL.
          CLEAR lt_shopping_cart_as.
          lt_shopping_cart_as = CORRESPONDING #( update-shoppingcart MAPPING FROM ENTITY ).
          LOOP AT update-shoppingcart  INTO DATA(shoppingcart) WHERE OrderUUID IS NOT INITIAL.
    *           select * from zashopcart_### where order_uuid = @shoppingcart-OrderUUID into @DATA(ls_shoppingcart) .
    *                      lt_shopping_cart_as = corresponding #( create-shoppingcart MAPPING FROM ENTITY ).

            MODIFY zashopcart_### FROM TABLE @lt_shopping_cart_as.
    *           endselect.
          ENDLOOP.
        ENDIF.
        IF zbp_shopcarttp_###=>mapped_purchase_requisition IS NOT INITIAL AND update IS NOT INITIAL.
          LOOP AT zbp_shopcarttp_###=>mapped_purchase_requisition-purchaserequisition ASSIGNING FIELD-SYMBOL(<fs_pr_mapped>).
            CONVERT KEY OF i_purchaserequisitiontp FROM <fs_pr_mapped>-%pid TO DATA(ls_pr_key).
            <fs_pr_mapped>-purchaserequisition = ls_pr_key-purchaserequisition.
    *        ZBP_SHOPCARTTP_###=>cv_pr_pid = <fs_pr_mapped>-%pid.
          ENDLOOP.
          LOOP AT update-shoppingcart INTO  DATA(ls_shopping_cart) WHERE %control-OverallStatus = if_abap_behv=>mk-on.
            " Creates internal table with instance data
            DATA(creation_date) = cl_abap_context_info=>get_system_date(  ).
    *      update zashopcart_### from  @( VALUE #(  purchase_requisition = ls_pr_key-purchaserequisition  pr_creation_date =  creation_date order_id = ls_shopping_cart-OrderID  ) ).
            UPDATE zashopcart_### SET purchase_requisition = @ls_pr_key-purchaserequisition, pr_creation_date = @creation_date WHERE order_uuid = @ls_shopping_cart-OrderUUID.
          ENDLOOP.
        ENDIF.
        LOOP AT delete-shoppingcart INTO DATA(shoppingcart_delete) WHERE OrderUUID IS NOT INITIAL.
          DELETE FROM zashopcart_### WHERE order_uuid = @shoppingcart_delete-OrderUUID.
          DELETE FROM zdshopcart_### WHERE orderuuid = @shoppingcart_delete-OrderUUID.
        ENDLOOP.
      ENDMETHOD.

      METHOD cleanup_finalize.
      ENDMETHOD.

    ENDCLASS.
    ```

   3. Save and activate.

    >**HINT:** The option **internal** can be set before the action name to only provide an action for the same BO. An internal action can only be accessed from the business logic inside the business object implementation such as from a determination or from another action.

   4. Go back to your behavior definition `ZR_SHOPCARTTP_###` and activate it again, if needed.

### Open documentation

You have 2 options to open the documentation inside ADT.

> **Option 1**:

>  1. Open your ABAP class **`zbp_shopcarttp_###`**, search for `i_purchaserequisitiontp`, press CTRL and click on it.
      ![service](docunew.png)

>  2. Now you are in the released object `i_purchaserequisitiontp`.
     Click **Open Documentation** to open it.
      ![service](docu2.png)

>  3. Now you are able to read the documentation.
      ![service](docunew2.png)

> **HINT:** You can also open the Element Info by clicking `i_purchaserequisitiontp` and pressing **`F2`**.
> ![service](docunew6.png)

> You can also switch to different layers inside the Element Info.
> ![service](docunew3.gif)

> **Option 2**:

> 1. Go back to tab `i_purchaserequisitiontp`. You are now able to see the behavior definition folder of the released object `i_purchaserequisitiontp`  in the project explorer. Now navigate to the documentation `i_purchaserequisitiontp` and open it.
      ![service](docunew4.png)
>**HINT**: You can also check the API State of released object and see its visibility by selecting the properties.
> 2. Now you can see the documentation.
      ![service](docunew5.png)

### Run SAP Fiori Elements preview

 1. Open the SAP Fiori preview, select your order entry and click **Create purchase requisition item**.

     ![preview](sb6.png)

 2. Now a purchase requisition item got created. Copy the purchase requisition item number for later use.

     ![preview](uinew.png)
 


### Check purchase requisition


 1. In the Project Explorer, select your system and right click on **Properties**.
 
     ![preview](systemnew.png)

 1. Select **ABAP Development** and copy the system URL without `-api`, paste it in a browser and **log in**.

     ![preview](systemnew2.png)

 2. Select the **Manage Purchase Requisitions** tile.

     ![preview](logonflp4.png)

 3. Search for your purchase requisition id and select your purchase requisition. 

     ![preview](sb8.png)

 4. Check the purchase requisition description. Your `CB-User ID` and your purchase requisition id should be listed here.

     ![preview](sb9.png)


### Test yourself
