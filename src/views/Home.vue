<template>
  <div class="home">
    <a href="https://github.com/portm/lier">Github</a>

    <div class="container">
      <div class="col">
        <h3>Lier</h3>
        <Editor mode='lier' :value='lierExample' @change='onLierEditorChange' />
      </div>

      <div class="col">
        <h3>JSON</h3>
        <Editor mode='javascript' :value='jsonExample' @change='onJsonEditorChange' />
      </div>
    </div>

    <h3>
      Validation:
    </h3>

    <pre v-if='htmlError' v-html="htmlError"></pre>
    <div v-if='error' class="red">
      {{error}}
    </div>
  </div>
</template>

<style lang="less" scoped>
.container {
  display: flex;

  .col {
    width: 50%;

    margin-right: 10px;
  }
}

.red {
  color: rgb(194, 85, 85);
}
</style>


<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Editor from '@/components/Editor.vue';
import * as lier from 'lier';
import inspect from 'lier/src/inspect';
import { browserify } from 'lier/src/brush';

@Component({
  components: {
    Editor,
  },
})
export default class Home extends Vue {
  public lierExample = lier.stringify(lier.parse(`
    type sf.a uint
    type sf.b str
    type B [
        {
            a: 1
        },
        int?,
        ...str
    ]
    type A {
        a: uint
        b: A[]
    }
    type Sub {
        a : sf.a | sf.b
        b : self.regex * self.regex + 1
        this : Sub[]
    }
    {
        @mock(1)
        regex : /^\\d$/
        /regex1/ : int
        # a
        # b
        oct : 066
        dec : 66
        hex : 0xff
        int : int
        str : str
        bool : bool
        optional? : bool
        byte : byte
        short : short
        uint : uint
        float : float
        @mock(2)
        @mock(1)
        enum : enum {
            # 1
            a = 1, b
        }
        @mock(3)
        allOf : int & uint
        anyOf : int | str
        # any 匹配任何东西
        any : any
        # never 不匹配任何东西
        never? : never
        @mockKey(1)
        $rest : any
        sub : Sub[]
        @mock(6)
        match : match self.regex {
            case 2 => 2 * 2
            case any => 3 * 2
        }
        sf: sf.a
        A: A
        B: B
        @range(10, 15)
        C?: int
    }
  `));

  public jsonExample = JSON.stringify(JSON.parse(`
    {
        "regex": 2,
        "oct": 54,
        "dec": 66,
        "hex": 255,
        "int": 1,
        "str": "1",
        "bool": true,
        "byte": 1,
        "short": 1,
        "uint": 1,
        "float": 1.1,
        "enum": 2,
        "allOf": 1,
        "anyOf": "1",
        "any": {},
        "a": 1,
        "sub": [
            {
                "a": 2,
                "b": 5,
                "this": [
                    {
                        "a": 2,
                        "b": 5,
                        "this": []
                    }
                ]
            }
        ],
        "match": 4,
        "sf": 1,
        "A": {
            "a": 1,
            "b": [
                {
                    "a": 1,
                    "b": []
                }
            ]
        },
        "B": [
            {
                "a": 1
            },
            1,
            "2"
        ]
    }
  `), null, 4);

  private lierValue = '';
  private jsonValue = '';
  private error = '';
  private htmlError = '';

  private onLierEditorChange(value: string) {
    this.lierValue = value;
    this.validate();
  }

  private onJsonEditorChange(value: string) {
    try {
      this.jsonValue = JSON.parse(value);
    } catch (err) {
      this.htmlError = '';
      this.error = err.message;
      return;
    }
    this.validate();
  }

  private validate() {
    try {
      this.error = '';

      const result = lier.validatex(this.jsonValue, this.lierValue);

      if (result) {
        this.htmlError = browserify(inspect(result));
      } else {
        this.htmlError = '<h3 style="color: #499c49">Valid!</h3>';
      }
    } catch (err) {
      this.htmlError = '';
      this.error = err.message;
    }
  }
}
</script>
