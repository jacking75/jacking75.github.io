---
layout: post
title: C# - Random 유틸리티
published: true
categories: [.NET]
tags: c# random .NET
---
.NET 라이브러리의 기본 Random에 기능을 추가한 것이다.  
  
  
아래 코드는 Unity용인데 조금 고치면 .NET에서도 사용할 수 있다.  
    
[출처](https://github.com/nubick/unity-utils/blob/master/sources/Assets/Scripts/Utils/RandomUtil.cs )   
```
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using Random = UnityEngine.Random;

namespace Assets.Scripts.Utils
{
    public static class RandomUtil
    {
        /// <summary>
        /// Return random bool value.
        /// </summary>
        public static bool NextBool()
        {
            return Random.Range(0, 2) == 0;
        }

        /// <summary>
        /// Return random item from item1 and item2 set.
        /// </summary>
        public static T Next<T>(T item1, T item2)
        {
            return NextBool() ? item1 : item2;
        }

        /// <summary>
        /// Return random item from item1, item2 and item3 set.
        /// </summary>
        public static T Next<T>(T item1, T item2, T item3)
        {
            int n = Random.Range(0, 3);
            return n == 0 ? item1 : (n == 1 ? item2 : item3);
        }

        /// <summary>
        /// Return random item from array.
        /// </summary>
        public static T NextItem<T>(T[] array)
        {
            return array[Random.Range(0, array.Length)];
        }

        /// <summary>
        /// Return random item from list.
        /// </summary>
        public static T NextItem<T>(List<T> list)
        {
            return list[Random.Range(0, list.Count)];
        }

        /// <summary>
        /// Return random enum item.
        /// </summary>
        public static T NextEnum<T>()
        {
            var values = Enum.GetValues(typeof(T));
            return (T)values.GetValue(Random.Range(0, values.Length));
        }

        /// <summary>
        /// Return random index of passed array. Index random selection is based on array weights.
        /// </summary>
        public static int NextWeightedInd(int[] weights)
        {
            int randomPoint = Random.Range(0, weights.Sum()) + 1;
            int sum = 0;
            for (int i = 0; i < weights.Length; i++)
            {
                sum += weights[i];
                if (randomPoint <= sum)
                    return i;
            }
            throw new Exception("Logic error!");
        }

        /// <summary>
        /// Return random index of passed array. Index random selection is based on array weights.
        /// </summary>
        public static int NextWeightedInd(float[] weights)
        {
            float randomPoint = Random.Range(0f, weights.Sum());
            float sum = 0f;
            for (int i = 0; i < weights.Length; i++)
            {
                sum += weights[i];
                if (randomPoint <= sum)
                    return i;
            }
            throw new Exception("Logic error!");
        }

        /// <summary>
        /// Return sub-list of random items from origin list without repeating.
        /// </summary>
        public static List<T> Take<T>(List<T> list, int count)
        {
            List<T> items = new List<T>();
            List<int> remainedIndexes = Enumerable.Range(0, list.Count).ToList();
            for (int i = 0; i < count; i++)
            {
                int selectedIndex = NextItem(remainedIndexes);
                remainedIndexes.Remove(selectedIndex);
                items.Add(list[selectedIndex]);
            }
            return items;
        }

        /// <summary>
        /// Shuffle list of items.
        /// </summary>
        public static void Shuffle<T>(this List<T> list)
        {
            for (int i = 1; i < list.Count; i++)
            {
                int indRnd = Random.Range(0, i + 1);
                T temp = list[i];
                list[i] = list[indRnd];
                list[indRnd] = temp;
            }
        }

        /// <summary>
        /// Shuffle array of items.
        /// </summary>
        public static void Shuffle<T>(T[] array)
        {
            for (int i = 1; i < array.Length; i++)
            {
                int indRnd = Random.Range(0, i + 1);
                T temp = array[i];
                array[i] = array[indRnd];
                array[indRnd] = temp;
            }
        }

        /// <summary>
        /// Return random point on line.
        /// </summary>
        public static Vector2 NextPointOnLine(Vector2 point1, Vector2 point2)
        {
            float t = Random.Range(0f, 1f);
            return new Vector2(Mathf.Lerp(point1.x, point2.x, t), Mathf.Lerp(point1.y, point2.y, t));
        }

        /// <summary>
        /// Return random point on line.
        /// </summary>
        public static Vector3 NextPointOnLine(Vector3 point1, Vector3 point2)
        {
            float t = Random.Range(0f, 1f);
            return new Vector3(Mathf.Lerp(point1.x, point2.x, t), Mathf.Lerp(point1.y, point2.y, t), Mathf.Lerp(point1.z, point2.z, t));
        }

        /// <summary>
        /// Get a chance with given percentage. If percentage is 25 it will return true each 4th time on an average.
        /// </summary>
        public static bool GetChance(int percentage)
        {
            return Random.Range(0, 100) + 1 <= percentage;
        }

        /// <summary>
        /// Gets a chance with give probability. If probability is 0.25 it will return true each 4th time on an average.
        /// </summary>
        public static bool GetChance(float probability)
        {
            return Random.Range(0f, 1f) < probability;
        }

        /// <summary>
        /// Get random normalized 2D direction as Vector2.
        /// </summary>
        public static Vector2 NextDirection()
        {
            return Random.insideUnitCircle.normalized;
        }

        /// <summary>
        /// Return Random.Range between two values which are stored in pairMinMax Vector2.
        /// Where pairMinMax.x is min value and pairMinMax.y is max value.
        /// </summary>
        public static float Range(Vector2 pairMinMax)
        {
            return Random.Range(pairMinMax.x, pairMinMax.y);
        }

        /// <summary>
        /// Return random point from rect bound (inside rect).
        /// </summary>
        public static Vector2 NextPointOnRect(Rect rect)
        {
            return new Vector2(Random.Range(rect.xMin, rect.xMax), Random.Range(rect.yMin, rect.yMax));
        }

        /// <summary>
        /// Return random point on rect border (perimeter of rect).
        /// </summary>
        public static Vector2 NextPointOnRectBorder(Rect rect)
        {
            float perimeterLength = (rect.width + rect.height) * 2f;
            float pointOnPerimeter = Random.Range(0f, perimeterLength);

            if (pointOnPerimeter < rect.width)//top border
                return new Vector2(rect.xMin + pointOnPerimeter, rect.yMax);

            pointOnPerimeter -= rect.width;

            if (pointOnPerimeter < rect.height)//right border
                return new Vector2(rect.xMax, rect.yMin + pointOnPerimeter);

            pointOnPerimeter -= rect.height;

            if (pointOnPerimeter < rect.width)//bottom border
                return new Vector2(rect.xMin + pointOnPerimeter, rect.yMin);

            pointOnPerimeter -= rect.width;

            //left border
            return new Vector2(rect.xMin, rect.yMin + pointOnPerimeter);
        }
    }
}
```
  
  
사용 예
```
using Assets.Scripts.Utils;
using System.Collections.Generic;
using UnityEngine;

public class Example : MonoBehaviour
{
    private enum Direction { Left, Right, Up, Down }

    public class Card { }

    private void Awake()
    {
        var randomBool = RandomUtil.NextBool();

        // 지정된 요소에서 랜덤한 요소를 반환한다
        var randomPerson = RandomUtil.Next( "me", "you" );

        // 배열에서 랜덤으로 요소를 반환
        var intArray = new[] { 1, 3, 5, 7, 9 };
        var randomInt = RandomUtil.NextItem( intArray );

        var list = new List<int> { 1, 3, 5, 7, 9 };
        randomInt = RandomUtil.NextItem( list );

        // 랜덤한 열거 타입의 요소를 반환
        var randomDirection = RandomUtil.NextEnum<Direction>();

        // 요소의 인덱스를 반환
        var weights = new int[] { 10, 10, 30, 50 };
        var randomIndex = RandomUtil.NextWeightedInd( weights );

        // 지정된 개수 랜던하게 요소를 반환
        var deckCards = new List<Card> { };
        var handCards = RandomUtil.Take( deckCards, 6 );

        list = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        RandomUtil.Shuffle( list );

        var array = new float[] { 1.1f, 1.2f, 1.3f, 1.5f, 1.6f };
        RandomUtil.Shuffle( array );

        // 랜덤한 한 점을 반환
        var point1 = new Vector2( 123f, 321f );
        var point2 = new Vector2( 0f, 0f );
        var randomPoint2 = RandomUtil.NextPointOnLine( point1, point2 );

        // 랜덤한 한 점을 반환 
        var point3 = new Vector3( 123f, 321f, 213f );
        var point4 = new Vector3( 0f, 0f, 0f );
        var randomPoint3 = RandomUtil.NextPointOnLine( point3, point4 );

        // 0 에서 100 까지의 확률에서 성공한지 어떤지 반환
        var percentage = 25;
        var isSuccess = RandomUtil.GetChance( percentage );

        // 0 .0 에서 1.0 까지의 확률에서 성공한지 어떤지 반환
        var probability = 0.25f;
        isSuccess = RandomUtil.GetChance( probability );
    }
}
```
  
