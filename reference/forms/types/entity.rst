EntityType Field
================

A special ``ChoiceType`` field that's designed to load options from a Doctrine
entity. For example, if you have a ``Category`` entity, you could use this
field to display a ``select`` field of all, or some, of the ``Category``
objects from the database.

+-------------+------------------------------------------------------------------+
| Rendered as | can be various tags (see :ref:`forms-reference-choice-tags`)     |
+-------------+------------------------------------------------------------------+
| Parent type | :doc:`ChoiceType </reference/forms/types/choice>`                |
+-------------+------------------------------------------------------------------+
| Class       | :class:`Symfony\\Bridge\\Doctrine\\Form\\Type\\EntityType`       |
+-------------+------------------------------------------------------------------+

.. include:: /reference/forms/types/options/_debug_form.rst.inc

Basic Usage
-----------

The ``entity`` type has just one required option: the entity which should
be listed inside the choice field::

    use App\Entity\User;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, [
        // looks for choices from this entity
        'class' => User::class,

        // uses the User.username property as the visible option string
        'choice_label' => 'username',

        // used to render a select box, check boxes or radios
        // 'multiple' => true,
        // 'expanded' => true,
    ]);

This will build a ``select`` drop-down containing *all* of the ``User`` objects
in the database. To render radio buttons or checkboxes instead, change the
`multiple`_ and `expanded`_ options.

.. _ref-form-entity-query-builder:

Using a Custom Query for the Entities
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to create a custom query to use when fetching the entities
(e.g. you only want to return some entities, or need to order them), use
the `query_builder`_ option::

    use App\Entity\User;
    use Doctrine\ORM\EntityRepository;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, [
        'class' => User::class,
        'query_builder' => function (EntityRepository $er): QueryBuilder {
            return $er->createQueryBuilder('u')
                ->orderBy('u.username', 'ASC');
        },
        'choice_label' => 'username',
    ]);

.. note::

    Using form collections may result in making too many database requests to
    fetch related entities. This is known as the *"N + 1 query problem"* and it
    can be solved by :ref:`joining related records <doctrine-associations-join-query>`
    when querying for Doctrine associations.

.. _reference-forms-entity-choices:

Using Choices
~~~~~~~~~~~~~

If you already have the exact collection of entities that you want to include
in the choice element, just pass them via the ``choices`` key.

For example, if you have a ``$group`` variable (passed into your form perhaps
as a form option) and ``getUsers()`` returns a collection of ``User`` entities,
then you can supply the ``choices`` option directly::

    use App\Entity\User;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, [
        'class' => User::class,
        'choices' => $group->getUsers(),
    ]);

.. include:: /reference/forms/types/options/select_how_rendered.rst.inc

Field Options
-------------

``choice_label``
~~~~~~~~~~~~~~~~

**type**: ``string``, ``callable`` or :class:`Symfony\\Component\\PropertyAccess\\PropertyPath`

This is the property that should be used for displaying the entities as text in
the HTML element::

    use App\Entity\Category;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('category', EntityType::class, [
        'class' => Category::class,
        'choice_label' => 'displayName',
    ]);

If left blank, the entity object will be cast to a string and so must have a ``__toString()``
method. You can also pass a callback function for more control::

    use App\Entity\Category;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('category', EntityType::class, [
        'class' => Category::class,
        'choice_label' => function (Category $category): string {
            return $category->getDisplayName();
        }
    ]);

The method is called for each entity in the list and passed to the function. For
more details, see the main :ref:`choice_label <reference-form-choice-label>` documentation.

.. note::

    When passing a string, the ``choice_label`` option is a property path. So you
    can use anything supported by the
    :doc:`PropertyAccess component </components/property_access>`

    For example, if the translations property is actually an associative
    array of objects, each with a ``name`` property, then you could do this::

        use App\Entity\Genre;
        use Symfony\Bridge\Doctrine\Form\Type\EntityType;
        // ...

        $builder->add('genre', EntityType::class, [
            'class' => Genre::class,
            'choice_label' => 'translations[en].name',
        ]);

``class``
~~~~~~~~~

**type**: ``string`` **required**

The class of your entity (e.g. ``App:Category``). This can be
a fully-qualified class name (e.g. ``App\Entity\Category``)
or the short alias name (as shown prior).

``em``
~~~~~~

**type**: ``string`` | ``Doctrine\Persistence\ObjectManager`` **default**: the default entity manager

If specified, this entity manager will be used to load the choices
instead of the ``default`` entity manager.

``query_builder``
~~~~~~~~~~~~~~~~~

**type**: ``Doctrine\ORM\QueryBuilder`` or a ``callable`` **default**: ``null``

Allows you to create a custom query for your choices. See
:ref:`ref-form-entity-query-builder` for an example.

The value of this option can either be a ``QueryBuilder`` object, a callable or
``null`` (which will load all entities). When using a callable, you will be
passed the ``EntityRepository`` of the entity as the only argument and should
return a ``QueryBuilder``. Returning ``null`` in the Closure will result in
loading all entities.

.. caution::

    The entity used in the ``FROM`` clause of the ``query_builder`` option
    will always be validated against the class which you have specified at the
    `class`_ option. If you return another entity instead of the
    one used in your ``FROM`` clause (for instance if you return an entity
    from a joined table), it will break validation.

Overridden Options
------------------

.. include:: /reference/forms/types/options/choice_name.rst.inc

In the ``EntityType``, this defaults to the ``id`` of the entity, if it can
be read. Otherwise, it falls back to using auto-incrementing integers.

.. include:: /reference/forms/types/options/choice_value.rst.inc

In the ``EntityType``, this is overridden to use the ``id`` by default. When the
``id`` is used, Doctrine only queries for the objects for the ids that were actually
submitted.

``choices``
~~~~~~~~~~~

**type**:  ``array`` | ``\Traversable`` **default**: ``null``

Instead of allowing the `class`_ and `query_builder`_ options to fetch the
entities to include for you, you can pass the ``choices`` option directly.
See :ref:`reference-forms-entity-choices`.

``data_class``
~~~~~~~~~~~~~~

**type**: ``string`` **default**: ``null``

This option is not used in favor of the ``class`` option which is required
to query the entities.

Inherited Options
-----------------

These options inherit from the :doc:`ChoiceType </reference/forms/types/choice>`:

.. include:: /reference/forms/types/options/choice_attr.rst.inc

.. include:: /reference/forms/types/options/choice_translation_domain_disabled.rst.inc

.. include:: /reference/forms/types/options/expanded.rst.inc

.. include:: /reference/forms/types/options/group_by.rst.inc

``multiple``
~~~~~~~~~~~~

**type**: ``boolean`` **default**: ``false``

If ``true``, the user will be able to select multiple options (as opposed
to choosing just one option). Depending on the value of the ``expanded``
option, this will render either a select tag or checkboxes if ``true`` and
a select tag or radio buttons if ``false``. The returned value will be a
Doctrine's Array Collection.

.. note::

    If you are working with a collection of Doctrine entities, it will be
    helpful to read the documentation for the
    :doc:`/reference/forms/types/collection` as well. In addition, there
    is a complete example in the :doc:`/form/form_collections` article.

.. include:: /reference/forms/types/options/placeholder.rst.inc

.. include:: /reference/forms/types/options/placeholder_attr.rst.inc

``preferred_choices``
~~~~~~~~~~~~~~~~~~~~~

**type**: ``array`` or ``callable`` **default**: ``[]``

This option allows you to move certain choices to the top of your list with a visual
separator between them and the rest of the options. This option expects an array
of entity objects::

    use App\Entity\User;
    use Symfony\Bridge\Doctrine\Form\Type\EntityType;
    // ...

    $builder->add('users', EntityType::class, [
        'class' => User::class,
        // this method must return an array of User entities
        'preferred_choices' => $group->getPreferredUsers(),
    ]);

The preferred choices are only meaningful when rendering a ``select`` element
(i.e. ``expanded`` false). The preferred choices and normal choices are separated
visually by a set of dotted lines (i.e. ``-------------------``). This can be customized
when rendering the field:

.. code-block:: twig

    {{ form_widget(form.publishAt, { 'separator': '=====' }) }}

.. include:: /reference/forms/types/options/choice_type_translation_domain.rst.inc

.. include:: /reference/forms/types/options/choice_type_trim.rst.inc

These options inherit from the :doc:`form </reference/forms/types/form>`
type:

.. include:: /reference/forms/types/options/attr.rst.inc

.. include:: /reference/forms/types/options/by_reference.rst.inc

.. include:: /reference/forms/types/options/data.rst.inc

.. include:: /reference/forms/types/options/disabled.rst.inc

.. include:: /reference/forms/types/options/empty_data_declaration.rst.inc

The actual default value of this option depends on other field options:

* If ``multiple`` is ``false`` and ``expanded`` is ``false``, then ``''``
  (empty string);
* Otherwise ``[]`` (empty array).

.. include:: /reference/forms/types/options/empty_data_description.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc

.. include:: /reference/forms/types/options/error_mapping.rst.inc

.. include:: /reference/forms/types/options/help.rst.inc

.. include:: /reference/forms/types/options/help_attr.rst.inc

.. include:: /reference/forms/types/options/help_html.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/label_attr.rst.inc

.. include:: /reference/forms/types/options/label_format.rst.inc

.. include:: /reference/forms/types/options/mapped.rst.inc

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/row_attr.rst.inc

.. include:: /reference/forms/types/options/label_translation_parameters.rst.inc

.. include:: /reference/forms/types/options/attr_translation_parameters.rst.inc

.. include:: /reference/forms/types/options/help_translation_parameters.rst.inc
